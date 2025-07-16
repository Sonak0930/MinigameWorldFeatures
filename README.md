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

## Pathfinding using Unity AI: NavMeshAgent

### Orbit-Rotation

There was a requirement for the vehicle which orbits around the center of the map with some distance.
It required to consideration:
- What is the direction of the vehicle for every time?
- How to match the rotation itself and overall orbit-rotation?

> 맵 중앙을 중심으로 빙글빙글 도는 탈것을 구현했습니다.
> '공전하는 탈것'은 몇가지 고려사항이 있었습니다.
> - 탈것이 매 프레임마다 보는 방향은 어디인가?
> - 탈것이 바라보는 방향과 공전 움직임을 어떻게 맞물리는가?


I tried to find a source code for rotation, but there was no resources left.

> 물체를 회전시키는 코드는, 개발노트에 코드를 첨부하지 않아서 찾지 못했고, 영상만 첨부했습니다.

- Result
https://github.com/user-attachments/assets/8505f4b9-a353-46a4-8a05-d277fce99d11


### NavMesh Pathfinding

<img width="1113" height="516" alt="image" src="https://github.com/user-attachments/assets/757b4aa5-e97c-4463-95aa-9d0830b97fc2" />

Instead of orbit rotation, The vehicle rotates around the circular path.
This should be different from previous approach: where rotation can be defined mathematically
The path can be ellipse, curved, or an arbitary spline.
Therefore, another approach which can be applied to all kinds of path was required
At the moment, I deciede to use the path as a pivot point where agent can patroll.

> 기존의 탈것 회전 아이디어가 길 위를 따라 회전하는 것으로 변경되었습니다.
> 이 길은 구불구불하거나, 타원 형태가 될 수도 있기 때문에 기존의 수학적인 경로는 사용할 수가 없었습니다.
> 그래서 UnityAI 라이브러리에 있는 NavMeshSurface를 이용해서, Agent가 경로를 찾을 수 있는 surface를 정의했습니다.

```
//C#
Start()
{
    for (let i = 0; i < this.way.transform.childCount; i++)
    {
        waypoints.push(this.way.transform.GetChild(i));
    }
    console.log("set dest");
    agent = this.GetComponent<NavMeshAgent>();

    this.UpdateDestination();
}
```

Start method initializes waypoints, which is used for the pivots of the agent path.
the pivot is defined by list of vectors which are the children of this object.

> Start 함수는 Agent의 Path에 사용될 waypoint를 정의합니다.
> path의 각 지점마다 child pivot을 지정해서, 이 오브젝트의 children으로 만들어놓았습니다.

```
//C#
Update()
{
    if (Vector3.Distance(this.transform.position, target) < 5)
    {
        console.log("move");
        this.IterateWaypointIndex();
        this.UpdateDestination();
    }
}

UpdateDestination()
{
    target = waypoints[waypointIndex].position;
    agent.SetDestination(target);
}

IterateWayPointIndex()
{
    waypointIndex++;
    if (waypointIndex == waypoints.length)
    {
        waypointIndex = 0;
    }
}

```

Update method keeps calculating the distance between our vehicle and next target position.
if the vehicle gets close enough to the target waypoints,
the waypoint is updated to next one.
This makes the vehicle to move along the path.

> 업데이트 함수는 탈것과 타겟까지의 거리를 계속해서 계산합니다.
> 만약 거리가 충분히 가까워지면, 탈것의 타겟을 다음 waypoint로 변경해서
> 탈것이 waypoint를 따라 천천히 순회하도록 만듭니다.

- Result

https://github.com/user-attachments/assets/3ba29780-2358-42a9-b813-ef52dba07eea

Raining and Smoke effect are also added for more details, using Particle System.

> 유니티 파티클 시스템을 활용해 비 내리는 효과와 매연 효과를 추가해서, 더욱 재미있게 연출했습니다.

