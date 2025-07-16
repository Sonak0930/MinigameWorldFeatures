# Zepeto World Features

This is a demo description of project features for Zepeto World.
Unfortunately, the project file cannot be attached due to a security issue.
The files are attached from the project.

> 이 프로젝트는 제페토 월드에 사용된 데모 기능들을 설명한 문서입니다.
> 대외비 문제로 인해 프로젝트 소스코드는 업로드 하지 못했습니다.
> 대신에 프로젝트에 사용했던 개발 문서와 사진, 영상자료를 첨부했습니다.
>


## Ranking System

<img width="342" height="372" alt="image" src="https://github.com/user-attachments/assets/f0ced3a9-d480-4cf4-9a2f-de199f45e8d7" />


[Source World: Kat-holic](https://world.zepeto.me/ko/detail/tmvNH3wK88CtafGfPxOYXLN?referrer=copylink_share)

There was a requirement to list the player's name and ID based on their game score.
The player can participate in each world and play the mini-games.
At the end, Players can check the leader board to see who recorded the highest score.
The source project is linked below.
So we used RankingBoard functions to query players' scores and names based on their ID.
and displayed them on the UI.

> 랭킹보드 시스템이 필요하다는 아이디어가 있었습니다.
> 미니게임 월드마다 게임을 수행하면 점수를 얻는데, 해당 점수로 랭킹 보드를 만들어서 유저들에게 보여주었습니다.
> 리더보드 덕분에 참여한 유저들이 누가누가 가장 높은 점수를 얻었는지 확인하고, 경쟁할 수 있도록 했습니다.
> 플레이어의 ID를 parameter로 Query를 하는 함수를 사용했습니다.

[Zepeto Studio: Searching Ranking Details](https://studio.zepeto.me/guides/searching-ranking-details)

```
//typescript

LeaderboardAPI.GetRangeRank(this.leaderboardID,this.startRank,this.endRank,this.resetRule,false
((result:GetRangeRankResponse)->{
    console.log(`result.isSuccess: `${result.isSuccess}`);
    if(result.rankInfo.myRank)
    {
        console.log(`my rank: ${result.rankInfo.myRank.rank.toString()}`);
        console.log(`member: ${result.rankInfo.myRank.member},
        rank:  ${result.rankInfo.myRank.rank},
        score: ${result.rankInfo.myRank.score},
        name:  ${result.rankInfo.myRank.name}`);
        var resultInfo=result.rankInfo;
        this.rank=resultInfo.myRank.rank.toString();
    }

}));
```

Using LeaderboardAPI.GetRangeRank function, you can queries user rank data from range of startRank to endRank(ex: 1st to 10th).
result info is printed on the console and used for later to display them on UI screen.

> LeaderBoard API의 GetRangeRank를 통해 특정 랭크 범위(1등~10등)의 플레이어 데이터를 받아올 수 있습니다.
> 콘솔을 통해서 출력하고, 랭킹보드 UI에 사용됩니다.

- Result
  


https://github.com/user-attachments/assets/105ee38d-9af5-48c6-b095-eee0c950591d



<br></br>

## Circular Orbit-Rotation of the vehicle.

There was a requirement for the vehicle which orbits around the center of the map with some distance.
It required to consideration:
- What is the direction of the vehicle for every time?
- How to match the rotation itself and overall orbit-rotation?

> 맵 중앙을 중심으로 빙글빙글 도는 탈것을 구현했습니다.
> '공전하는 탈것'은 몇가지 고려사항이 있었습니다.
> - 탈것이 매 프레임마다 보는 방향은 어디인가?
> - 탈것이 바라보는 방향과 공전 움직임을 어떻게 맞물리는가?



