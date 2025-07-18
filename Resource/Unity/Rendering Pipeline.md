## URP (Universal Render Pipeline)

- **목적**:  
    다양한 플랫폼(모바일, PC, 콘솔 등)에서 **균형 잡힌 성능과 그래픽 품질**을 제공하는 범용 렌더링 파이프라인
    
- **특징**:
    
    - 가볍고 최적화가 잘 되어 있어서 저사양 기기에서도 잘 동작
        
    - 모바일, VR, 저사양 PC 게임에 적합
        
    - 셰이더, 라이팅, 포스트 프로세싱이 HDRP보다 단순하지만 효율적
        
    - 커스터마이징과 확장성 좋음
        
- **주로 쓰이는 곳**:
    
    - 모바일 게임
        
    - VR/AR 콘텐츠
        
    - 저~중사양 PC 게임
        
    - 크로스 플랫폼 프로젝트
        

---

## HDRP (High Definition Render Pipeline)

- **목적**:  
    **고품질 그래픽과 사실적인 시각 효과**를 위해 설계된 렌더링 파이프라인
    
- **특징**:
    
    - 최신 PC, 콘솔 등 고성능 하드웨어에 최적화
        
    - 물리 기반 렌더링(PBR), 고급 라이팅(실시간 레이트레이싱 등) 지원
        
    - 복잡한 셰이더와 효과, 고해상도 그림자, 반사, 전역 조명(Global Illumination) 등 가능
        
    - 성능 요구가 높아서 저사양 기기에는 적합하지 않음
        
- **주로 쓰이는 곳**:
    
    - AAA급 PC/콘솔 게임
        
    - 영화, 시네마틱, 건축 시각화 등 고퀄리티 그래픽 프로젝트