---
title: 2D 플랫포머 게임
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/portfolio_banner_24_01.png 
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/portfolio_banner_24_01.png 
---

2024년 1월 2D 플랫포머 개인 프로젝트

[깃허브](https://github.com/mob954325/2DPlatformer_1)

## 기간
개발기간 : 24.01.07 - 24.01.28

## 목표
2D 게임 개발 이해 및 플래포머 게임 개발

### 세부 목표
1. Input System을 이용한 플레이어 움직임 및 공격
2. 범위 내 들어오면 공격하는 포탑
3. 플레이어, 보스 체력 UI
4. Dialog로 간단한 대화 표현

### 플레이 Gif

게임 기본 플레이  
<img src="https://github.com/mob954325/2DPlatformer_1/assets/87255621/7c1dae3b-95c4-4731-a24e-dcfa0a9ff93e" alt="2D Platformer Game Play Gif">  
  
보스전  
<img src="https://github.com/mob954325/2DPlatformer_1/assets/87255621/2677ae7b-31a5-47cd-bb4f-4d32e12ad1d1" alt="2D Platformer Game Boss Play Gif">  

### 주요 코드

**Input System** 

```
void Update()
{
    // 업데이트 문에서 인풋에 따른 오브젝트 움직임
    transform.Translate(Time.deltaTime * _inputMove * _moveSpeed);

    // ...
}

private void OnMove(InputAction.CallbackContext context)
{
    // Input System 에서 인풋값 받기
    _inputMove = context.ReadValue<Vector2>();
    // ...
}

```

**범위 내 들어오면 공격하는 포탑**  

```
/// <summary>
/// 레이케스트를 이용한 플레이어 감지
/// </summary>
void CheckPlayer()
{
    _shotInterval = new WaitForSeconds(_shotIntervalTime);
     // 원점, 지름, 방향, 길이, 감지할 Layer
    _rayTarget = Physics2D.CircleCast(transform.position, _scanRange, Vector2.zero, 0, _targetMast);

    Debug.DrawRay(transform.position, Vector2.right * _scanRange, Color.red);

    if (_rayTarget.collider != null && _rayTarget.collider.gameObject.CompareTag("Player"))
    {
        _target = _rayTarget.collider.gameObject.transform; // player
        _bulletDirection = _target.transform.position - transform.position;

        // 총알 생성
        if (!_isShot) StartCoroutine(Shot());
    }
}

```