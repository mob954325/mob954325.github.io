---
title: 다이노런
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/portfolio_banner_24_09.png 
---

2D 횡스크롤 게임인 다이노 런 개발

[깃허브](https://github.com/mob954325/Unity_Dinorun)

## 기간
24.09.04 - 24.09.11

## 목표
2D 횡스크롤 게임인 다이노 런 개발

### 세부 목표
사운드 매니저 만들기

### 메뉴 및 시작 GIF
![DinoStart](https://github.com/user-attachments/assets/61b35172-ab5b-4465-b5c4-6b3b001e3416)

### 사망 및 메뉴 복귀 GIF 
![Dino_Dead](https://github.com/user-attachments/assets/08e077d5-f8c1-4b8b-9084-ad2ac5cbd78f)

### 주요 코드

사운드 매니저 코드

```
public enum AudioType
{
    Button,
    Jump,
    Land,
    Die
}

public class SoundManager : MonoBehaviour
{
    private AudioSource audioScoure;

    /// 실행할 오디오 파일들
    public AudioClip[] audioClips;

    private void Awake()
    {
        audioScoure = GetComponent<AudioSource>();
    }

    /// 오디오 플레이 함수
    /// <param name="type">플레이할 타입</param>
    public void PlayAudio(AudioType type)
    {
        audioScoure.Stop();

        if(audioScoure.clip != audioClips[(int)type])
        {
            audioScoure.clip = audioClips[(int)type];
        }

        audioScoure.Play();
    }

    /// 오디오 플레이 함수 (int형)
    /// <param name="typeInt">플레이 할 타입의 int형</param>
    public void PlayAudio(int typeInt)
    {
        PlayAudio((AudioType)typeInt);
    }
}
```  
  
플레이어 점프에서의 오디오 사용
```
/// 캐릭터 점프 함수
protected virtual void Jump()
{
    if(!isGround) return; 

    rigid.AddForce(Vector2.up * jumpPower, ForceMode2D.Impulse);
    isGround = false;

    anim.SetBool(isJumpToHash, !isGround);
    manager.soundManager.PlayAudio(1); // 점프 오디오 실행
}
```