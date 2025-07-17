# Zepeto World Features

This is a demo description of project features for Mini-game World.
Unfortunately, the project file cannot be attached due to a security issue.
The files are attached from the project.

> 이 프로젝트는 미니게임 월드에 사용된 데모 기능들을 설명한 문서입니다.
> 대외비 문제로 인해 프로젝트 소스코드는 업로드 하지 못했습니다.
> 대신에 프로젝트에 사용했던 개발 문서와 사진, 영상자료를 첨부했습니다.
>

[text](#ranking-system)

[text](##navmesh-pathfinding)

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
//C# based script(ZepetoScript)

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

### Orbit Rotation

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
- 
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


## Exploring the whole map with bird-view camera

<img width="640" height="406" alt="image" src="https://github.com/user-attachments/assets/dc8226e2-1226-4564-b799-c9ccc02817d7" />

There was a telescope which player can interact with to explore the whole map.
The Player camera started to move along the path while getting tilted.
The camera follows the pre-defined pivots in a very smooth movement.

> 플레이어들이 맵을 한 눈에 담을 수 있도록, 망원경을 구현했습니다.
> 망원경의 움직임은 미리 지정한 pivot을 따라 움직이고, 부드러운 곡선을 그리도록 만들었습니다.

```
//C#
Init()
{
    for (let i = 0; i < this.pivotParent.transform.childCount; i++)
    {
        var childtransform = this.pivotParent.transform.GetChild(i);
        pivots.push(childtransform);

        this.transform.position - pivots[0].position;
        this.transform.rotation = pivots[0].rotation;
    }
}
Update() {
    if (!this.stop)
        this.MoveCamera();
}
```

This process is similar to NavPathfinding, using pre-defined pivots in a children list.

> 위에서 vehicle 회전에 사용한 것처럼 미리 피벗을 생성해 children으로 넣어줬습니다.

```
//C#
MoveCamera() 
{
    if (pivots == undefined)
    {
        this.Init();
        console.log("index " + index);
        var distance = Vector3.Distance(this.transform.position, pivots[index].position);

        //if it gets close enough,
        if (distance < 10)
        {
            if (index + 1 == pivots.length)
            {
                //clear index
                index = 0;
            }
            else
            {
                index += 1;
            }
        }
        else
        {
            var nextPos = Vector3.Slerp(this.transform.position, pivots[index].position, this.slerpSpeed);
            var nextRot = Quaternion.Slerp(this.transform.rotation, pivots[index].rotation, this.slerpSpeed);
            this.transform.position = nextPos;
            this.transform.rotation = nextRot;
        } 
    }
}

```

The distintive point is that it interpolates between to pivots using Slerp(Spherical Linear Interpolation) 
for both of position and rotation.

> 특이점은 Position과 Rotation을 보간할 때 선형(Linear)이 아니라, Spherical Linear Interpolation을 사용합니다.

<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/c52e53e0-cd8a-4eef-9a57-ad7d5996e6a4" />

- Path using Lerp
Camera path has some sharp corners when transition to next pivot location.


> - Lerp를 사용한 경로
> 카메라의 경로가 뚝뚝 끊기는 것처럼 부자연스럽습니다.

<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/e6c5ae12-1190-4760-b96f-bef8c1a7e740" />

- Path using Slerp
Camera shows more smooth transition for position and rotation.

> - Slerp를 사용한 경우
> 훨씬 더 부드러운 곡선 경로를 그리는 것을 볼 수 있습니다.


## Teleportation Synchronization

This was the most challenging part, which I spent the majority of my time.
I have tested this feature over and over again, because there was no appropriate solution or official documentation
to describe this issue in detail.

> 텔레포트 동기화는 제가 가장 많은 시간을 투자한 기능입니다.
> 동기화 문제에 대한 구체적인 솔루션이나 공식 문서가 없었기 때문에, 꽤나 애를 먹었습니다.

```
//Trigger Code
// This code is attached to the trigger box for teleportation.

OnTriggerEnter(coll:Collider)
{
    ZepetoPlayers.instance.LocalPlayer.zepetoPlayer.character.Teleport(
        pivot.transform.position,
        pivot.transform.rotation
    );

    //Custom script for teleport
    //this method generates position and rotation data
    //and send it to the server for synchronization.
    cs.customTeleport(pivot.transform.position, pivot.transform.rotation)
}
```

### Trigger Box code 

This is the code to move the local Player, which is visible on the client side.
There is no problem, but when another player joins, he might not see you teleport. 
Rather, your character is stuck in that position, or suddenly starts to run to the teleport destination.

> 이 코드는 Local Player를 텔레포트하는 함수입니다.
> Local Player는 Client, 내 화면에서 보이는 캐릭터를 지칭합니다.
> 내가 보기에는 정상적으로 이동하지만, 다른 캐릭터가 나를 봤을 때는 내 캐릭터가 움직이지 않거나,
> 갑자기 텔레포트 방향으로 뛰어가는 문제가 있었습니다.

```
//ClientStarter
public customTeleport(p:UnityEngine.Vector3, r:UnityEngine.Quaternion)
{
    //when this room object is not null
    //room is created when player joins in the game.
    if (this.room)
    {
        //Initialize room position data
        const data = new RoomData();
        const pos = new RoomData();

        pos.Add("x", p.x);
        pos.Add("y", p.y);
        pos.Add("z", p.z);

        data.Add("position", pos.GetObject());


        const rot = new RoomData();
        rot.Add("x", r.x);
        rot.Add("y", r.y);
        rot.Add("z", r.z);

        //WARNING_NOT_VERIFIED
        //rot.Add("w",r.z);
        data.Add("rotation",rot.GetObject());

        //Send Packet to the server
        this.room.Send("onTeleport", data.GetObject());
    }
}
```

### ClientStarter for teleportation

CustomTeleport has two roles.
- Generate Room Data for position and rotation.
- Send the packet with the room data.

I omitted w (rotation angle in radians) when initialization of the rot.

> CustomTeleport의 기능은 다음과 같습니다.
> - 위치와 쿼터니언 기반 Room Data 생성
> - 해당 데이터 패킷을 서버에 보내기.
> 작성하다보니 w를 빼먹었는데, quaternion은 x,y,z,w 4개 parameter를 가지고 있습니다.
> 그중에 w는 rotation angle(라디안)을 나타냅니다.
```
//Client Starter
this.multiplay.RoomJoined += (room:Room)=>
{
    room.OnStateChange += this.OnStateChange;
    room.AddMessageHandler("ChangePlayerTransform",(message:Transform)=>
    {
        var global = ZepetoPlayer.instance.GetPlayer(message.clientid).character;
        var local = ZepetoPlayer.instance.LocalPlayer.ZepetoPlayer.character;

        //WARNING: NOT VERIFIED CODE
        var pos = this.ParseVector3(message.position);
        var rot = 
            new UnityEngine.Quaternion(message.rotation.x,
            message.rotation.y,
            message.rotation.z,
            1);
        /*
        var rot = this.ParseVector3(message.position),
            new UnityEngine.Quaternion(message.rotation.x,
            message.rotation.y,
            message.rotation.z,
            message.rotation.w);
        */    
        global.Teleport(pos,rot);
        global.transform.SetPositionAndRotation(pos,rot);


        local.Teleport(global.transform.position, global.transform.rotation);
        
        //WARNING: NOT RELIABLE CODE
        if (UnityEngine.Vector3.op_Eqaulity(global.transform.position, local.transform.position))
        {
            local.Teleport(global.transform.position, global.transform.rotation);
        }
        
        /*
        Instead of using Equality function, Calculate the min distance between Global and Local position.
        When local is within the distance, perform teleportation.
        
        */
    });
};
```

### Client Starter: RoomJoined Event

RoomJoined Event is registered to the local player when the player first joins the room(Server).
The room got handler for "ChangePlayerTransform" that is called when the server broadcasts that packet.
The process of broadcasting is described below.

> RoomJoined 이벤트는 플레이어가 게임에 접속하면 호출됩니다.
> 해당 이벤트를 통해 플레이어가 접속했을때 특정 기능을 실행할 수 있습니다.
> RoomJoined 이벤트에서는 플레이어 접속시 ChangePlayerTransform에 대한 Handler를 부여합니다.
> 그러면 서버가 해당 이름의 패킷을 발송했을 때 캐치를 해서 이벤트를 수행할 수 있습니다.
> 서버 브로드캐스팅 과정은 아래의 Index.server 코드에 명시되어 있습니다.
 
When the packet is received, it gets a reference to the Global and Local players.
Then it extracts the position and rotation from the message.
They are applied to the Teleport function.

> 패킷이 도착하면 가장 먼저 Global Player와 Local Player reference를 생성합니다.
> 그리고 패킷에 있는 position과 rotation 정보를 추출하고, 이를 바탕으로 Teleportation을 수행합니다.

Teleportation happens a third time.
- First: Global Character is teleported.
- Second: Local Player is teleported
- Compare the Global and local positions and match their positions.

> 텔레포트는 사실 3번에 걸쳐 일어납니다.
> - 1번째: 글로벌 캐릭터를 텔레포트 합니다.
> - 2번째: 로컬 플레이어를 텔레포트 합니다.
> - 3번째: 글로벌과 로컬 플레이어의 포지션을 비교해 로컬플레이어 포지션을 업데이트 합니다.

```
//Index
//This handler is in ServerSide.
this.onMessage("onTeleport", (client, message) =>
{
    //query player data based on the sessionId
    const player = this.state.players.get(client.sessionId);

    //Generate a container to hold pos and rot
    const transform = new Transform();
    transform.position = new Vector3(
        message.position.x,
        message.position.y,
        message.position.z
    );

    //WARNING: NOT_VERIFIED
    //transform.rotation might be quaternion not a EulerAngle.
    transform.rotation = new Vector3(
        message.rotation.x,
        message.rotation.y,
        message.rotation.z
    );

    /*
    transform.rotation = new Quaternion(
        message.rotation.x,
        message.rotation.y,
        message.rotation.z,
        message.rotation.w
    );
    */

    //Update the server player based on the sessionID of the client.
    transform.clientId = client.sessionId;
    plater.transform = transform;

    //Broadcast ChangePlayerTransform packet to all players in the server.
});
```

### Index.server code

Index.server is a server code that broadcasts the packets to all clients.
This means changes to the packet information are applied to all of the local players.
It is crucial to update the local player position when teleportation, to synchronize the local player's transform and
Server Player Transform.

> Index.server파일은 모든 클라이언트들에게 패킷을 보내는(broadcast)역할을 합니다.
> 이 과정을 통해 서버에 저장된 해당 플레이어 정보가 다른 로컬 플레이어들에게도 동일하게 전달이 됩니다.
> 만약 server player와 local player가 다르다면, 두 플레이어가 서로 바라본 위치가 다르게 되고, 멀티플레이에 지장이 생기게 됩니다.


<img width="404" height="409" alt="image" src="https://github.com/user-attachments/assets/68560d2c-bbd6-463d-bd78-ddbe4b7e88dc" />

### Schema settings for packet data type

This schema defines the input type for the packet.

> 이 스키마를 통해 패킷에 사용될 인풋 타입을 지정할 수 있습니다.

## Boat Buoyancy Simulation

<img width="749" height="235" alt="image" src="https://github.com/user-attachments/assets/f1e7aa0f-82a1-4ef5-8fb8-f9b5fb87f40b" />

A custom boat that the player can control, and it fluctuates when it is dropped from a high point.

> 커스텀 보트를 구현해 사용자가 조이스틱으로 움직일 수 있게 만들었습니다.
> 높이차를 계산해서 높은 곳에서 떨어질 경우 물에 잠겼다가 출렁입니다.

```

Update()
{
    //Calculate the height of the boat compared to the surface height
    //this.transform.position.y: current boat height
    //this.boatHeight = surface height
    this.CalculateHeight();
    var gravity;

    //CASE1: Boat is below the surface-> going upward
    if (this.transform.position.y < this.boatHeight)
    {
        
        var displacementMultiplier = Mathf.Clamp(
            (this.boatHeight - this.transform.position.y) / this.depthBeforeSubmerged, 0, 1)
            * this.displacementAmount;

        gravity = new Vector3(0, Mathf.Abs(Physics.gravity.y) * displacementMultiplier, 0);

        this.rigidBody.AddForce(gravity, ForceMode.Acceleration);
    }

    //CASE2: Boat is above the surface-> going down(no additional force)
    else
    {
        var dispalcementMultiplier = Mathf.Clamp(
            (this.boatHeight - this.transform.position.y) / this.depthBeforeSubmerged, 0, 1)
            * this.displacementAmount;

        //gravity is not applied.
    }
}
```

### Calculate Boat Height


Gravity Behaviour is determined by the difference between the current boat height and the surface height
- boatHeight(Surface Height) > transform.y(boat y-coordinates) -> The boat is floating
- boatHeight(Surface Height) < transform.y(boat y-coordinates) -> The boat is submerging.
  
<img width="623" height="399" alt="image" src="https://github.com/user-attachments/assets/196f7aa9-1279-48a4-9f30-71eb31956992" />

> 보트의 중력은 수면과 보트의 위치를 비교해 결정됩니다.
> - 보트가 수면 아래에 있다 -> 보트가 위로 올라감
> - 보트가 수면 위에 있다 -> 보트가 내려감 (추가적인 힘 필요 없음.)

The interesting point is that the magnitude of the force depends on the distance between the surface and the boat.
Usually, when the boat becomes closer to the surface, the force becomes weaker.
The height of the fluctuation decreases over time.

> 특이점은 보트가 수면에 가까워질수록, 이 힘의 크기가 줄어든다는 것입니다.
> 그래서 시간이 지나면 보트가 위 아래로 진동하다가 진폭이 점점 줄어듭니다.

```
CalculateHeight()
{
    if (this.raycastPivot == null)
    {
        //WARNING: NOT RELIABLE CODE
        this.raycastPivot = this.vehicle.transform.GetChild(2).gameObject;
    }

    //DebugRay
    Debug.DrawRay(this.raycastPivot.transform.positioon, Vector3.down);
    
    //Raycast to the surface
    let hit = $ref<RaycastHit>();
    if (Physics.Raycast(this.raycastPivot.transform.position, Vector3.down, hit, 10))
    {
        let hitInfo =$unref(hit);
        var obj = hitInfo.collider.gameObject;
        if (obj.tag == "Wave")
        {
            this.SetBoatHeight(obj.transform.position.y);
        }
    }
}
```

### Raycasting from the bottom of the boat to the water surface.

It displays debug rays to visualize the direction of the raycasting.
Raycasting only works for the mesh filter object, which is tagged "Wave"

> 보트의 바닥면에서 아래 방향으로 debug ray를 그립니다.
> 이를 통해 플레이어는 게임모드에서 레이 캐스팅을 확인할 수 있습니다.
> 바닥면은 반드시 wave tag가 되어있어야 하며, mesh filter가 있어야 raycasting으로 찾을 수 있습니다.



