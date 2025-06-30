- **유니티 C# 스크립트를 통해 캡처 설정 및 시작 가능**:
    
    - `AvProMovieCapture` C# 스크립트가 캡처의 모든 고수준 제어(시작, 중지, 설정 변경)를 담당함
        
- **준비사항**
    
    - ** `PrepareCapture()` 메서드와 같은 곳에서 캡처 시작 전 다양한 사전 검사를 수행. 
    - **녹화 설정**: 유니티에 미리 설정해놨던 녹화 설정을 네이티브 플러그인에 정수형태로 전달하여 녹화기 생성 함수 호출
        
        - `NativePlugin.CreateRecorderVideo`와 같은 함수 호출 시 매개변수로 모든 설정값(해상도, 코덱, 오디오 등)이 전달. 그리고 이 호출의 결과로 **`_handle`이라는 정수 ID**를 받음
            
        - **세부 확인 사항 (예시)**:
            
            - **에디터, 플러그인 기능 유효성 검사**: `ValidateEditionFeatures()`
                
            - **파일 존재 여부 확인 및 삭제**: `File.Exists()`, `File.Delete()`
                
            - **디스크 공간 확인**: `IsEnoughDiskSpace()`, `Utils.DriveFreeBytes()`
                
            - **프레임 레이트 조절, V-Sync 비활성화 등**: `Time.captureDeltaTime`, `QualitySettings.vSyncCount` 조정.
                
            - **모션 블러 및 오디오 캡처 컴포넌트 설정**: `MotionBlur` 및 `UnityAudioCapture` 컴포넌트 준비.
                
            - **코덱 확인**: `_selectedVideoCodec`이 null이 아닌지 확인.
                
            - **로그 구성**: `StringBuilder`를 사용하여 로그 메시지 준비.
                
- **매 프레임마다 렌더 텍스처로 화면을 캡처해 GPU 포인터를 가져와 네이티브 플러그인에 전달**:
    
    - **로직**
        
        - `UpdateTexture()` (C#) 함수에서 카메라의 `targetTexture`를 `_target` `RenderTexture`로 **일시적으로 변경**하고 렌더링을 지시하여, `_target`에 현재 프레임 이미지를 **갱신**.
            
        - 갱신된 `_target` `RenderTexture`의 **GPU 메모리 포인터**를 `_target.GetNativeTexturePtr()`로 가져옴.
            
        - 이 포인터를 `NativePlugin.SetTexturePointer()`를 통해 네이티브 플러그인에 **전달하여 세팅**.
            
        - 그리고 `GL.IssuePluginEvent()`를 통해 **네이티브 플러그인에 실제 캡처 작업(`RenderCaptureEventFunction`)을 실행하라고 명령**. (이때 `PluginID | (int)renderEvent | handle` 형태의 압축된 ID를 전달.)
            
- **네이티브 플러그인 내부에서 받은 설정, 포인터를 활용해 픽셀 데이터를 받아 인코딩, 멀티플렉싱, 디스크에 파일 생성, 세션 관리 등을 진행함**:
    
    - ** `GL.IssuePluginEvent`에 의해 호출되는 네이티브 플러그인 콜백 함수(예: `RenderCaptureEventFunction`)가 이 모든 작업을 수행.
        
        - `_handle`을 통해 해당 녹화 세션(`RecordingSession` 객체)을 찾아, `CreateRecorderVideo` 호출 시 저장되었던 **모든 설정값에 접근**.
            
        - `SetTexturePointer`로 받은 **GPU 포인터**를 활용하여 GPU 메모리에서 **픽셀 데이터(프레임 이미지)를 읽어옴.** (이 과정은 GPU-to-CPU Readback이거나 GPU 내 하드웨어 인코딩일 수 있습니다.)
            
        - 읽어온 데이터를 **인코딩**(비디오 및 오디오 코덱 사용).
            
        - 인코딩된 비디오/오디오 스트림을 **멀티플렉싱**하여 하나의 영상 컨테이너 파일로 생성.
            
        - 이 데이터를 **디스크에 파일로 생성**하고 저장.
            
        - 전반적인 **캡처 세션 상태 관리**와 오류 처리도 진행.