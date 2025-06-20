**멀티플레이어 재접속 클라이언트 안전한 스텝 동기화 구현**

- 재접속 클라이언트에 방의 진행 상황을 전송하는 도중 발생할 수 있는 스텝 꼬임 문제를 해결하기 위해, 조건 검사를 통한 RPC 지연 실행 큐를 설계 및 구현
    
- `UniTask` 기반 비동기 큐 처리로 게임 상태가 특정 조건에 도달할 때까지 RPC 호출을 대기시켜, 진행 중인 스텝과의 충돌 방지
    
- 이를 통해 네트워크 동기화 안정성을 확보하고, 게임 진행 상황 일관성 유지가능



```
// 네트워크 재접속 클라이언트 동기화를 위한 RPC 대기 큐 구현

using Cysharp.Threading.Tasks;
using System;
using System.Collections.Generic;

public static class NetworkPending
{
    private class RpcEntry
    {
        public Action RpcAction;
        public Func<bool> ConditionCheck;
    }

    private static readonly Queue<RpcEntry> _rpcQueue = new();
    private static bool _isProcessing = false;

    /// <summary>
    /// RPC 실행 요청 등록
    /// </summary>
    /// <param name="rpcAction">실행할 RPC 액션</param>
    /// <param name="conditionCheck">RPC 실행 전 대기할 조건 검사 함수</param>
    public static void RegisterRpc(Action rpcAction, Func<bool> conditionCheck)
    {
        _rpcQueue.Enqueue(new RpcEntry
        {
            RpcAction = rpcAction,
            ConditionCheck = conditionCheck
        });

        ProcessQueue().Forget();
    }

    /// <summary>
    /// 큐에 등록된 RPC를 순차적으로 처리
    /// 조건이 만족될 때까지 대기 후 실행
    /// </summary>
    private static async UniTaskVoid ProcessQueue()
    {
        if (_isProcessing) return;
        _isProcessing = true;

        while (_rpcQueue.Count > 0)
        {
            var entry = _rpcQueue.Peek();

            // 조건이 만족될 때까지 대기
            await UniTask.WaitUntil(() => entry.ConditionCheck?.Invoke() == true);

            // 조건 만족 후 RPC 실행 및 큐에서 제거
            _rpcQueue.Dequeue().RpcAction?.Invoke();

            await UniTask.Yield();
        }

        _isProcessing = false;
    }
}

```