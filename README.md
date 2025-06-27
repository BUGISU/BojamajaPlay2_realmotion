# 프로젝트 기술 설계서: Bojama PLAY2 RealMotion

## 🎮 보자마자 PLAY2 리얼모션

![대표 이미지](https://github.com/JISUSAMA/JISUSAMA/assets/38304918/4d110036-8117-4be2-b888-8f69a7c1e388)

Leap Motion을 활용해 **손 제스처 기반 모션 게임 콘텐츠**로 구성된 Unity 프로젝트입니다.  
모바일 게임 "보자마자 PLAY2"의 게임 구조를 기반으로, 실시간 손동작 입력을 활용한 10개의 미니게임을 리얼모션 콘텐츠로 재탄생시켰습니다.

- 각 게임은 30초 동안 진행되며, 클래식 모드 / 랭킹 모드 두 가지 방식으로 플레이 가능합니다.
- 순차적인 미션 구조, 손 동작 튜토리얼, 랭킹 등록 시스템, 별점 시스템, 사운드 및 이펙트 처리까지 포함된 종합형 콘텐츠입니다.

---

## 📅 개발 기간

- **2021년 1월 ~ 2021년 8월 (8개월)**
- 팀원 3인: 기획 1, 아트 1, 개발 1
- **리얼모션 게임 구조 전체 개발 및 시스템 설계 100% 담당**

---

## 🛠️ 기술 스택 및 구조

| 영역        | 기술                           | 설명 |
|-------------|--------------------------------|------|
| 게임 엔진   | Unity3D (C#)                   | 전체 콘텐츠 구조 설계 및 씬 기반 시스템 구성 |
| 모션 인식   | Leap Motion Controller         | 손 인식 및 제스처 판단 (엄지/검지/소지) |
| UI 처리     | Unity UI, DOTween              | 애니메이션, 타이머, 슬라이더, 패널 전환 |
| 사운드 관리 | AudioSource / Manager 패턴     | 상황별 효과음 및 배경음 통합 관리 |
| 데이터 저장 | PlayerPrefs                    | 점수, 별점, 총점, 닉네임 기록 저장 |
| 씬 전환     | SceneManager + 커스텀 로직     | 10개 게임 순차 실행, 종료 시 결과 씬 이동 |

---

## 🧩 시스템 흐름 요약

```

▶ 메인 모드 선택
├ 클래식 모드: 원하는 게임 1개 선택
└ 랭킹 모드: 10개 게임 자동 진행
└ 점수 합산 → 랭킹 등록 → 랭킹 조회
▶ 게임 실행 (30초 제한)
└ 손 제스처 튜토리얼 + 타이머 + 점수 처리
▶ 결과 화면 (별점, 점수 시각화)

````

---

## 🧠 주요 스크립트 구조 및 기능

### 🎮 GameManager.cs

- 게임 모드 분기, 다음 게임 자동 진행, 종료 씬 전환 담당

```csharp
public void NextScenes()
{
    if (RankMode)
    {
        if (RandListCountCheck == 9) LastGameCheck = true;
        SceneCheck(); // 다음 게임 씬 로드
        RandListCountCheck += 1;
    }
}
````

---

### ⏱ Timer.cs / Countdown.cs

* 제한 시간 30초 처리 및 시작 카운트다운 연출

```csharp
IEnumerator StartTimer()
{
    timeLeft = 30f;
    while (timeLeft > 0f)
    {
        timeLeft -= Time.deltaTime;
        UpdateUI();
        yield return null;
    }
    EndTimer(); // 시간 초과 처리
}
```

---

### ⭐ Score.cs

* 점수 누적, 별점 계산, 슬라이더 UI 업데이트 처리

```csharp
public void UpdateStarRating()
{
    starPick = (score > highscore) ? 5 : (score / Min) + 1;
}
```

---

### 📊 DataManager.cs

* 현재 점수 및 별점을 PlayerPrefs에 저장

```csharp
public void SaveDataScore()
{
    if (GameManager.RankMode)
        PlayerPrefs.SetInt("GameTotalScore", TotalScore + scoreManager.score);
}
```

---

### 🧼 Punisher.cs

* 점수 감소 시 오버레이와 빨간 텍스트로 시각적 경고 연출

```csharp
IEnumerator _ExecutePenalty()
{
    _penaltyOverlay.alpha = 1;
    while (_penaltyOverlay.alpha > 0.1f)
    {
        _penaltyOverlay.alpha -= Time.deltaTime * 7f;
        yield return null;
    }
}
```

---

## 🎮 게임별 콘텐츠 구성

| 게임 이름      | 키워드        | 사용 스크립트 예시                       |
| ---------- | ---------- | -------------------------------- |
| 먼지를 털어라    | 충돌 삭제, 난이도 | `Dusty.cs`, `MobSpawner.cs`      |
| 전골을 끓여라    | 드래그, 컨베이어  | `Food.cs`, `Conveyor.cs`         |
| 샌드위치를 만들어라 | 순서 조립, 주문서 | `Sandwich.cs`, `OrderManager.cs` |
| 장난감을 조립하라  | 부품 매칭, 흔들림 | `ToyPart.cs`, `ConveyorBelt.cs`  |
| 좀비를 막아라    | 자동조준, 슈팅   | `Gun.cs`, `ZombieSpawner.cs`     |
| 괴물새를 쫓아라   | 타이밍, 반응성   | `Eagle.cs`, `MBirdRealMotion.cs` |
| 조각상을 만들어라  | 다단계 타격     | `Statue.cs`, `StonDestroy.cs`    |
| 밸브를 잠가라    | 상태 변화, 클릭  | `Valve.cs`, `Pipe_ValveGrup.cs`  |
| 청기백기를 올려라  | 지시어 해석     | `FlagManager.cs`, `Flag.cs`      |
| 피자 반죽을 던져라 | 회전, 크기 인식  | `PizzaDough.cs`, `Leaflet.cs`    |

---

## 🔈 사운드/이펙트 시스템

* `SoundManager.cs`에서 전역 재생 제어
* 각 게임 이벤트(성공/실패/타격)마다 전용 사운드 클립 호출

```csharp
public void PlayStar()
{
    sfxPlayer.PlayOneShot(clip_Star);
}
```

* `StarPanelAble.cs`와 연계하여 별점 도달 시 자동 재생

---

## 🏆 랭킹 시스템

* `RankRegistration.cs`: 닉네임 등록, 중복 체크
* `HighRankerManager.cs`: 상위 랭커 목록 서버 조회
* `MyRankingManager.cs`: 내 랭킹 결과 확인
* `RankModManager.cs`: 총합 점수 합산 후 UI 출력

```csharp
PlayerPrefs.SetInt("GameTotalScore", totalScore + newScore);
```

---

## 📺 튜토리얼 / 제스처 안내

* `GuideLine.cs`, `HandsGesture.cs`에서 손 위치 시각화
* 제스처는 손가락 3개 (엄지/검지/소지) 조합으로 판별됨
* 게임 시작 시 `TutorialPanel` → `StartCountdown` 흐름으로 진행

---

## ✅ 핵심 성과 요약

| 항목      | 내용                     |
| ------- | ---------------------- |
| 총 개발 기간 | 8개월                    |
| 전체 게임 수 | 10종                    |
| 주요 입력   | Leap Motion 손 제스처      |
| 핵심 시스템  | 점수/별점 시스템, 튜토리얼, 랭킹 등록 |
| 스크립트 수  | 약 80개, 클래스 기반 구조       |
| UI 구조   | 약 40개 화면/팝업/패널 구성      |

---

## 🎬 홍보 영상

* [📺 시즌1 영상](https://www.youtube.com/watch?v=wDnnUCvHxpY)
* [📺 시즌2 영상](https://www.youtube.com/watch?v=ZHqRtNmgdR4)

---

**“보자마자 PLAY2 리얼모션”은 기존 모바일 콘텐츠를 Leap Motion 기반의 몰입형 게임으로 재구성하며,
모션 인식 게임 설계와 실시간 상호작용 콘텐츠 개발 경험을 쌓을 수 있었던 프로젝트입니다.**


