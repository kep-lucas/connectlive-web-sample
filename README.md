# **WebSDK**

# **소개**
안녕하세요. ConnectLive는 화상 및 음성을 교환하는 서비스를 쉽고 빠르게 개발할 수 있도록 지원하는 플랫폼입니다.

일반적인 인터넷을 통한 화상 및 음성통화 서비스는 서비스에 특화된 구성 요소 외에 미디어 서버, 미디어를 다루는 전용 네트워크 장비, 미디어 품질의 확보 등, 고려해야할 사항과 기술이 많이 존재합니다. ConnectLive 플랫폼은 서버 API, 미디어 서버, 화상 통화를 위한 인프라를 모두 제공하고 있으며, 플랫폼 사용자는 이들에 대해 알지 못 해도 자신이 원하는 서비스를 손쉽게 구현할 수 있습니다.

미디어 서버는 화상 통화 서비스에서 가장 중요한 역할을 수행하는 요소입니다. 미디어 서버는 참여자의 각 미디어를 서로 교환할 수 있도록 중계하고 품질을 관장하며 발화자의 음성을 제어해 혼란을 제어합니다. 또한, 녹화와 방송을 위한 기능도 제공합니다. ConnectLive는 뛰어난 미디어 서버 기술력으로 국내 유일하게 자체 개발한 미디어 서버를 제공하고 있습니다.


# **WebSDK Getting Started**
## **설치**
아래와 같이 첨부한 javascript를 html에 포함할 수 있습니다.

```
//최신버전
<script src="https://cdn.jsdelivr.net/npm/@connectlive/connectlive-web-sdk/dist/connectlive-web-sdk.min.js"></script>

//특정버전
https://cdn.jsdelivr.net/npm/@connectlive/connectlive-web-sdk@0.0.7/dist/connectlive-web-sdk.min.js
```

이렇게 script태그로 삽입된 sdk는 ConnectLive객체를 통해 사용할 수 있습니다.

```
const conf = ConnectLive.createConference();
```

npm을 통해 설치할 수도 있습니다.
```
npm install @connectlive/connectlive-web-sdk
```

ES6 모듈 패턴으로 사용할 수 있습니다.
```
import ConnectLive from '@connectlive/connectlive-web-sdk';
```

## **간단한 화상회의 구현하기**
이제 준비를 마쳤으니 간단한 화상회의를 구현해 보도록 하겠습니다. 순서대로 따라 오시면 됩니다.

1. HTML 문서 작성하기
```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width,initial-scale=1.0">
        <title>ConnectLive 화상회의</title>
    </head>
    <body>
        <div class="local-container"></div>
        <div class="remote-container"></div>
    </body>
</html>
```


2. signIn API를 통해 서비스키 인증하기

발급 받은 서비스 키에 대한 인증을 수행합니다. 

```
//promise 사용
ConnectLive.signIn({
    serviceId: '서비스아이디',
    serviceKey: '서비스키',
    secret: '시크릿키'
}).then(()=>{
    alert('인증에 성공했습니다.');
}).catch(()=>{
    alert('인증에 실패했습니다.');
});
```


3. 로컬 미디어 생성하기

로컬 오디오를 위해 createLocalMedia를 제공하고 있습니다. createLocalMedia를 통해 객체를 생성하면, 각각 video, audio 속성에 접근할 수 있습니다.

그리고 이들 객체의 attach라는 API를 통해 비디오 엘리먼트 연결해 화면에 로컬 비디오를 재생할 수 있습니다.

```
//미디어 생성방법
ConnectLive.createLocalMedia({
    audio: true,
    video: true
}).then((localMedia)=>{
    //방법 1
    const video = localMedia.video.attach();
    document.querySelector('.local-container').appendChild(video);

    //방법 2
    localMedia.video.attach(document.getElementById('video-tag-id'));

    //방법 3
    document.getElementById('video-tag-id').srcObject = localMedia.video.getMediaStream();
});
```

4. 컨퍼런스 객체 생성하고 접속하기
ConnectLive Web SDK는 화상회의를 위한 API를 제공합니다. 화상회의를 위한 객체를 생성하고 connect API를 이용해 룸에 접속할 수 있습니다. 

```
const conf = ConnectLive.createConference();

conf.connect('룸 아이디');
```


5. 오디오, 비디오 공유하기

나의 오디오와 비디오를 공유하려면 publish메소드를 사용합니다.

```
conf.publish([ localMedia ]);
```


6. 상대방의 비디오 구독하기

상대방의 비디오를 구독하기 위해서는 subscribe 와 몇가지 이벤트를 사용해야합니다. 여기서는 간단한 화상회의 구현을 위해 connected 이벤트와 remoteVideoPublished 이벤트만 소개합니다. connected이벤트는 룸에 접속하면 호출되는 이벤트이고 remoteVideoPublished 이벤트는 상대방이 룸에 들어오고 자신의 미디어를 공유하면 호출됩니다.

룸에 접속하면 호출되는 connected 이벤트는 기존 참여자를 담고 있는 participants를 인자로 받습니다. 여기서는 기존 참여자들의 비디오를 구독합니다.

```
conf.on('connected', (evt)=>{
    //기존 참여자 배열을 순회하며 비디오 구독 및 엘리먼트 생성

    evt.participants.forEach((participant) => {
        const unsubscribeVideos = participant.getUnsubscribedVideos();
        conf.subscribe([unsubscribeVideos[0].videoId]).then((remoteVideos)=>{
            const video = remoteVideos[0].attach();
            document.querySelector('.remote-container').appendChild(video);
        });
    });
});
```


먼저 룸에 접속하고 나중에 새로운 참여자가 들어와 자신의 미디어를 공유하면 remoteVideoPublished 이벤트가 호출됩니다. 여기에서 상대방의 비디오를 구독하고 비디오 엘리먼트를 생성할 수 있습니다.

```
conf.on('remoteVideoPublished', (evt)=>{
    conf.subscribe([evt.remoteVideo.videoId]).then((remoteVideos)=>{
        const video = remoteVideos[0].attach();
        document.querySelector('.remote-container').appendChild(video);
    });
});
```

# **상세 가이드**
## **Local Media**
사용자 컴퓨터의 오디오와 비디오 장치에 접근할 수 있는 방법을 제공합니다. 기본 사용법은 아래와 같습니다.
```
ConnectLive.createLocalMedia({
    audio: true,
    video: true
}).then((localMedia)=>{
    //장치에 성공적으로 접근했을 때
})
```

promise 패턴외에 await를 사용할 수도 있습니다.
```
const localMedia = await ConnectLive.createLocalMedia(({
    audio: true,
    video: true
});
```

생성된 로컬 미디어video, audio 속성을 가지고 있습니다.
비디오의 경우는 다음 3가지 방법으로 비디오 태그와 연결할 수 있습니다.

```
// 방법 1. attach를 이용해 새로운 엘리먼트를 생성한다.
const video = localMedia.video.attach();
document.querySelector('.local').appendChild(video);

// 방법 2. attach에 비디오 엘리먼트를 전달한다.
document.querySelector('.local').appendChild(document.elementById('비디오태그아이디'));

// 방법 3. 기존 엘리먼트에 미디어스트림을 직접 연결한다.
document.getElementById('video-tag-id').srcObject = localMedia.video.getMediaStream();
```
위 방법 중 1, 2번은 detach()로 연결을 해제 할 수 있습니다.

```
localMedia.video.detach();
```


### **Local 장치 목록 가져오기**
```
//오디오 장치 가져오기
await localMedia.getMicDevices();

//비디오 장치 가져오기
await localMedia.getCameraDevices();

//스피커 장치 가져오기
await localMedia.getSpeakerDevices();
```

장치 목록을 가져오는 API는 Promise객체를 반환하며 다음과 같이 사용할 수 있습니다.
```
localMedia.getCameraDevices().then((videoDevices)=>{
    console.log(cameraDevices);

    \*
        [{
            deviceId: "8d1f5a537f0f2ae93e610734b053f10385df93d3ffddf33e47faa758109aa531"
            groupId: "fbb0bd92666880c78bec7ff6dd683d56af51e14ac34322b883c65882789c005b"
            kind: "videoinput"
            label: "Snap Camera"
        }, {
            deviceId: "5b99e9442edc7ea16d6209e1e3164ee458eb7771e62b396156d972ff3314d4c3"
            groupId: "5efce94d3f5483f5e5dee3efb637f3ba3916e7b9bdfcf59052eaa763ebc871d9"
            kind: "videoinput"
            label: "FaceTime HD 카메라(내장형) (05ac:8514)"
        }]
    */
});
```

await를 사용할 수도 있습니다.
```
const cameraDevices = await localMedia.getCameraDevices();
```

### **비디오 장치 변경하기**
switchCamera로 접근 중인 비디오 장치를 변경할 수 있습니다. 예를 들어 전면 카메라를 사용하다 후면 카메라로 변경할 수 있습니다. 만약 switchCamera시 이미 화상회의를 진행중이라면, 기존 비디오 영상을 제거하고 새로운 카메라로 연결을 바로 이어 갑니다.

```
localMedia.switchCamera('디바이스아이디').then(()=>{
    // localMedia.video.attach()등을 통해 새로 비디오 엘리먼트를 연결해 비디오를 재생할 수 있습니다.
});
```
await를 사용할 수도 있습니다.

```
await localMedia.switchCamera('디바이스아이디');

//switchCamera호출 다음에 localMedia.video.attach()등을 통해 새로 비디오 엘리먼트를 연결해 비디오를 재생할 수 있습니다.
```


### ** HD 모드 적용하기
publish 전에 video의 품질을 좋게 하기 위해 HD 모드를 적용할 수 있습니다.
```
localMedia.video.setHd(true);
```


## **화상회의**
ConnectLive Web SDK를 이용한 화상회의 서비스 구현에 대해 보다 자세히 알아보도록 하겠습니다. 화상회의 서비스를 위해서는 우선 ConnectLive.createConference()를 통해 화상회의 객체를 선언해야 합니다.

```
const conf = ConnectLive.createConference({
    //options
});
```

이 때 옵션을 넘길 수 있습니다. createConference가 취하는 옵션은 다음과 같습니다.

|**이름**|**기본값**|**설명**|
| :-: | :-: | :-: |
|videoReceiverInitialCount|10|초기 영상 리시버 개수|
|videoReceiverGrowthRate|10|영상 리시버가 부족한 경우 증가 단위|
|videoReceiverMaximumCount|50|최대 영상 리시버 개수|


여기서 리시버라는 용어가 등장합니다. 리시버란, 비디오를 수신하기 위한 통로입니다. 각각의 비디오를 수신하기 위한 통로이기에 10개의 비디오를 수신하고 싶다면 10개 이상의 리시버가 존재해야 합니다. 

videoReceiverInitialCount는 초기에 몇 개의 리시버를 만들어 둘지 여부를 나타냅니다. 기본값인 10을 그대로 사용하고 참여자가 1명 들어와서 비디오를 구독(subscribe) 했다면, 남은 리시버는 9개가 됩니다. 그리고 후에 참여자가 나가게되면 다시 리시버는 다시 10개가 됩니다. 또 다른 예를 들어보겠습니다. 이번에는 한 명의 참여자가 자신의 얼굴이 나오는 비디오와 화면 공유 비디오까지 두개를 공유했다면, 이를 구독 사람은 리시버 2개 사용하고 남은 리시버는 8개가 됩니다.

이번에는 10명의 사용자가 각각 자신의 비디오를 공유 했고 11명째가 들어와서 자신의 비디오를 공유했다면, 이를 구독하는 사람은 videoReceiverGrowthRate 옵션을 이용해 리시버의 갯수를 총 20개로 늘리게 됩니다. 그리고 11번째 들어온 참여자의 비디오를 11번째 리시버에 할당해 총 리시버는 20개, 남은 리시버는 9개가 됩니다.

그리고 이러한 행위를 videoReceiverMaximumCount에 도달할때까지 수행합니다. 


### **미디어 공유하기**
자신의 미디어를 공유하려면 publish API를 사용합니다. 다음과 같이 로컬 미디어 객체를 publish API의 인자로 전달하기만 하면 됩니다.

```
conf.publish([ localMedia ]);
```

### **미디어 구독하기**
상대방의 비디오를 구독하려면 subscribe API를 사용합니다. 

```
const remoteVideos = await conf.subscribe([videoId]);

const video = remoteVideos[0].attach();

document.querySelector('.remote-container').appendChild(video);
```

여기서 인자로 넘겨지는 비디오 아이디는 무엇일까요? 이것은, 상대방이 공유한 비디오의 고유한 아이디값입니다. 이 비디오 아이디를 배열로 subscribe에 전달하면 리모트 비디오 배열을 반환합니다. 반환되는 리모트 비디오 객체 attach 메소드를 제공하며 이를 통해 video 태그에 바인딩하면 상대방 비디오에 대한 구독이 완료됩니다.

그럼 상대방 비디오 아이디는 어디서 얻을 수 있을까요? 그건 참여자 객체의 getUnsubscribedVideos()를 통해 알 수 있습니다. 이는 참여자의 현재 구독중이지 않은 리모트 비디오 배열을 반환합니다. 

```
conf.on('connected', (e)=>{
    //기존 참여자 배열을 순회하며 비디오 구독 및 엘리먼트 생성
    e.participants.forEach(async (participant) => {
        const unsubscribedVideos = participant.getUnsubscribedVideos();
        const remoteVideos = await conf.subscribe([unsubscribedVideos[0].videoId]);
        const video = remoteVideos[0].attach();
        document.querySelector('.remote-container').appendChild(video);
    });
});
```

```
conf.on('remoteVideoPublished', (e)=>{
    const remoteVideos = await conf.subscribe([e.video.videoId]);
    const video = remoteVideos[0].attach();
    document.querySelector('.remote-container').appendChild(video);
});
```

### **미디어 공유 해제하기**
자신의 비디오 공유를 해제하려면 unpublish API를 사용합니다. 공유과 마찬가지로 로컬 비디오 배열 객체를 받습니다.

```
await conf.unpublish([ localMedia ]);
```

상대방이 자신의 비디오를 공유해제했다면 이는 remoteVideoUnpublished이벤트로 연결됩니다. 해당 이벤트 내에서 상대방의 비디오를 제거합니다.

```
conf.on('remoteVideoUnpublished', (evt)=>{
    evt.remoteVideo.detach();
});
```

### **미디어 구독 해제하기**
상대방의 비디오 구독을 해제하려면 unsubscribe API를 사용합니다. 구독과 마찬가지로 비디오 아이디를 받습니다.

```
await conf.unsubscribe([videoId]);
```

### **오디오를 점유하고 있는 참여자 표시하기**
ConnectLive 화상회의는 오디오를 최대 4개까지만 제공하고 있습니다. 이는 곧 오디오를 점유하고 있는 참여자의 오디오를 4개까지만 재생한다는 것입니다. 그 이상의 오디오 재생은 혼란을 증가 시켜 실제 알아들을 수 없는 대화가 됩니다. 오디오를 점유하고 있다고 해서 실제로 말을 하고 있다는 뜻은 아닙니다. 오디오를 점유하지만 해당 참여자도, 다른 참여자도 모두 말을 하고 있지 않다면. 오디오 점유는 그대로 유지됩니다.

그럼 오디오를 차지하고 있는 참여자는 어떻게 표시할까요?

이는 remoteAudioSubscribe와 remoteAudioUnsubscribe이벤트로 알 수 있습니다. 이는 참여자가 오디오를 점유하면 그리고 오디오 점유가 해지되면 호출 됩니다.

```
conf.on(‘remoteAudioSubscribed’, evt => {
    evt.participant
});
```

```
conf.on(‘remoteAudioUnsubscribed’, evt => {
    evt.participant
});
```

### **각 참여자의 오디오 레벨 가져오기**
오디오를 점유하고 있는 참여자와는 별도로 각 참여자의 오디오 레벨을 가져올 수 있습니다. 오디오 레벨을 통해 실제 말하고 있는 참여자에 대한 표시를 할 수 있습니다. 아래 메서드를 주기적으로 호출해 오디오 레벨을 가져옵니다.

```
conf.getRemoteAudioLevels();
\*
{
    remoteParticipants: [{
        remoteParticipant: RemoteParticipant,
        level: 0.00630957344480193
    }]
}
*/
```

### **수신 중인 비디오의 품질 변경하기**
리모트 비디오 객체에는 setQuality라는 메소드를 제공하고 있습니다. 해당 메소드는 인자로 “l”, “m”, “h”라는 인자 중 하나를 받습니다. 각각 low, middle, high를 뜻합니다. 
high는 하나만 유지됩니다. 만약 A, B 비디오를 순서대로 high 지정했다면 마지막으로 지정한 B만 high로 설정됩니다.

```
const remoteVideo = remoteParticipant.getVideo(‘비디오 아이디’)

remoteVideo.setQuality(‘h’);
```

### **이벤트**
위에서 소개한 API외에 화상 회의 객체는 다음과 같은 이벤트를 더 제공하고 있습니다.

|**이름**|**인자**|**설명**|
| :-: | :-: | :-: |
| connecting | progress: number | 연결과정에서 각 과정이 지날때마다 호출됩니다. 연결 과정은 0 ~ 100%까지로 표현됩니다. |
| connected |{ participants: [] } | 룸에 성공적으로 접속하면 호출 되는 이벤트 입니다. 인자로 기존 접속하고 있는 participants 목록이 인자로 전달됩니다. |
| participantEntered |{ participant: {} } | 새로운 참여자가 들어오면 호출 되는 이벤트 입니다. 참여자 객체를 인자로 전달됩니다. |
| participantLeft |{ participantId: '' } | 참여자가 룸에서 떠나면 호출되는 이벤트입니다. 나간 참여자 아이디가 인자로 전달됩니다. |
| localMediaPublished | localMedia: LocalMedia | 자신의 로컬 미디어를 공유하면 호출되는 이벤트입니다. |
| localMediaUnpublished | localMedia: LocalMedia | 자신의 로컬 미디어를 공유 해제하면 호출되는 이벤트입니다. |
| remoteVideoPublished | remoteParticipant: RemoteParticipant, remoteVideo: RemoteVideo | 상대 참여자가 비디오를 공유하면 호출됩니다. 인자로 참여자 객체와 비디오 객체가 전달됩니다. 비디오객체 안의 비디오 아이디를 이용해 해당 비디오를 구독하고 비디오 태그에 비디오를 바인딩 할 수 있습니다.|
| remoteVideoUnpublished | remoteParticipant: RemoteParticipant, remoteVideo: RemoteVideo | 상대 참여자가 비디오 공유를 해제하면 호출됩니다. 인자로 참여자 객체와 비디오 객체가 전달됩니다. 비디오 객체를 이용해 비디오 태그에 비디오를 해제할 수 있습니다. |
| remoteAudioPublished | remoteParticipant: RemoteParticipant, remoteAudio: RemoteAudio | 상대 참여자가 오디오를 공유하면 호출됩니다. 인자로 참여자 객체와 오디오 객체가 전달됩니다. |
| remoteAudioUnpublished | remoteParticipant: RemoteParticipant, remoteAudio: RemoteAudio | 상대 참여자가 오디오 공유를 해제하면 호출됩니다. 인자로 참여자 객체와 오디오 객체가 전달됩니다. |

| remoteAudioSubscribe | remoteParticipant: RemoteParticipant, remoteAudio: RemoteAudio | 4개의 오디오 중 하나를 참여자가 점유하면 호출 되는 이벤트입니다. |
| remoteAudioUnsubscribe | remoteParticipant: RemoteParticipant, remoteAudio: RemoteAudio | 점유하고 있던 오디오가 해제되면 호출되는 이벤트 이다. 해당 사용자가 말을 하지 않고 있다는 것을 의미한다. |

| remoteAudioStateChanged | remoteParticipant: RemoteParticipant, remoteAudio: RemoteAudio | 다른 참여자의 오디오의 상태 alwaysOn 또는 enabled가 변경되면 호출됩니다. alwaysOn이란, 오디오를 계속적으로 점유해서 나의 오디오가 항시 전달되도록 하는 기능입니다. LocalAudiod의 setAlwaysOn 메소드로 이를 설정할 수 있습니다.

enabled을 false로 설정할 경우, 오디오를 잠시 mute 결과를 얻을 수 있습니다. LocalAudio의 setEnabled 메소드로 이를 설정할 수 있습니다. |

| remoteVideoStateChanged | remoteParticipant: RemoteParticipant, remoteVideo: RemoteVideo | 다른 참여자의 비디오 상태 곧 enabled 상태가 변경되면 호출됩니다. enabled을 false로 설정할 경우, 비디오를 잠시 mute 결과를 얻을 수 있습니다. LocalVideo의 setEnabled 메소드로 이를 설정할 수 있습니다. |

| disconnected |  | 컨퍼런스 룸에서 정상적으로 나오면 호출되는 이벤트. |
| error | 에러 메시지 | 미디어서버와의 연결이 실패했을 경우, 호출됩니다. 해당 에러 이벤트에서 초기화면으로 이동한다든가 하는 행위가 필요합니다. |

## **화면 공유하기**
ConnectLive는 프리젠테이션을 위한 화면 공유를 지원합니다. 화면 공유용 미디어 객체를 얻기 위해서는 createLocalScreen API를 이용 합니다.

```
const localScreen = await ConnectLive.createLocalScreen({
    audio: true,
    video: true
});
```
화면 공유용 객체 선언시 audio옵션을 줄 수 있습니다. 이 경우, 예를 들어 공유한 크롭 탭에서 현재 오디오가 재생되고 있다면 해당 오디오까지 공유되어집니다.
화면 공유용 비디오 객체를 생성했다면, 해당 객체가 화면 공유를 위한 비디오임을 표시해야 합니다. ConnectLive 트랙 객체는 setExtraValue라는 API를 제공합니다. 이는 해당 트랙에 원하는 형태의 문자열 값을 추가할 수 있고 이 값을 참여자들과 공유할 수 있습니다. 여기에 화면 공유용 트랙임을 명시합니다. 여기서는 screen이라는 문자열 값을 명시했지만 원하는 어떠한 값이든 상관없습니다.

```
localScreen.video.setExtraValue('screen');
```

이를 공유하는 방법은 기존 publish API 를 그대로 이용합니다.
```
conf.publish( [ localScreen ] );
```


이제 상대방이 공유한 화면 공유 비디오를 구독해야 합니다. 구독은 역시 connected이벤트와 remoteVideoPublished이벤트와 subscribe API를 그대로 이용합니다.

```
conf.on('connected', (e)=>{
    //기존 참여자 배열을 순회하며 비디오 구독 및 엘리먼트 생성
    e.participants.forEach(async (participant) => {
        const unsubscribeRemoteVideos= participant.getUnsubscribeVideos();
        const remoteVideos = await conf.subscribe([unsubscribeRemoteVideos[0].videoId]);
        const video = remoteVideos[0].attach();
        document.querySelector('.remote-container').appendChild(video);
    });
});
```

```
conf.on('remoteVideoPublished', (evt)=>{
    const remoteVideos = await conf.subscribe([ evt.remoteVideo.videoId ]);
    const video = remoteVideos [0].attach();
    document.querySelector('.remote-container').appendChild(video);
});
```


화면 공유 비디오는 서비스마다 다르게 보일 수 있습니다. 예를 들어, 화면 공유 비디오를 가운데 크게 보이길 원한다거나 혹은 모든 참여자가 공유한 화면 공유 비디오를 바둑판식으로 배열할 수도 있습니다. 그러려면 다른 일반 비디오와 구분해야할 필요가 있습니다. 위 코드에서 participant.getUnsubscribeVideos()와 e.remoteVideo 에 주목해 보면 이들이 리모트 비디오 객체를 반환한다는 것을 알 수 있습니다. 리모트 비디오 객체는 videoId 외에 extraValue라는 값을 전달합니다. 이 값을 이용해 상대방이 공유한 화면 공유 비디오를 구분 할 수 있습니다.

```
conf.on('connected', (evt)=>{
    //기존 참여자 배열을 순회하며 비디오 구독 및 엘리먼트 생성
    evt.participants.forEach(async (participant) => {
        const unsubscribeRemoteVideos= participant.getUnsubscribeVideos();
        if(unsubscribeRemoteVideos[0].extraValue === 'screen') {
            //화면 공유
            const remoteVideos = await conf.subscribe([unsubscribeRemoteVideos[0].videoId]);
        } else {
            //일반 비디오
        }
    });
});
```


```
conf.on('remoteVideoPublished', (evt)=>{
    if(evt.remoteVideo.extraValue === 'screen') {
        //화면 공유
        const remoteVideos = await conf.subscribe([evt.remoteVideo.videoId]);
    } else {
        //일반 비디오
    }
});
```

## **녹화하기**
ConnectLive는 본인의 화면을 녹화할 수 있습니다. 기본 사용 방법은 아래와 같습니다.
```
await conf.publish(([localMedia], true);
```
위와 같이 publish 메서드의 두번째 인자를 true로 지정합니다. 만약 이미 local media를 publish하고 있다면 unpublish후 publish를 다시 진행합니다.
```
await this.conf.unpublish([this.localMedia]);

await this.conf.publish([this.localMedia], true);
```
녹화 결과는 웹훅으로 전달됩니다.



## **에러 처리**
Connectlive 에러 객체는 두가지 타입으로 나눠집니다. ServerError, ClientError 타입입니다. 이름에서도 알 수 있듯이, 각각 서버 에러와 클라이언트 에러에 대한 정보를 담고 있습니다.
에러 객체는 code와 메시지 속성을 지니고 있으며 해당 에러를 통해 어떤 에러인지 구분할 수 있습니다.

** 서버 에러
|**name**|**message**|**Description**|
| :- | :- | :- |
| -11001 | Scheme mismatch | 해당 service id에 설정된 값과 입력된 auth. scheme이 다름 |
| -11002 | Unauthorized | 허가되지 않은 요청에 대한 응답 |
| -11003 | Invalid service id or key | 등록되지 않은 service id 또는 service key |
| -11004 | Forbidden | 인증 확인 실패 |
| -11005 | Invalid token | 현 토큰 확인 실패 |


** 클라이언트 에러
|**name**|**message**|**Description**|
| :- | :- | :- |
| 0 | 메시지 | 클라이언트 에러 메시지 |


### **서비스 인증 에러처리**
인증 과정에서의 에러는 아래와 같이 처리할 수 있습니다.

```
//promise
ConnectLive.signIn({
    serviceId: '서비스아이디',
    serviceKey: '서비스키',
    secret: '시크릿키',
}).catch((err)=>{
    //에러 처리
});
```

```
//await
try{
    await ConnectLive.signIn({
        serviceId: '서비스아이디',
        serviceKey: '서비스키',
        secret: '시크릿키',
    });
} catch (err) {
    //에러 처리
}
```

### **connect시 에러 처리**
룸에 접속하기 위해 호출하는 connect API는 룸에 정상적으로 접속하지 못 하면 에러를 반환합니다.

```
//promise
conf.connect('룸 아이디').catch((err)=>{
    //에러 처리
});
```

```
//await 사용시
try{
    await conf.connect('룸 아이디');
} catch () {
    //에러 처리
}
```

### **미디어 공유 에러 처리**
미디어를 공유하기 위해 호출하는 publish API도 미디어 공유에 실패하면 에러를 반환합니다.

```
//promise
conf.publish( [ localMedia ] ).catch((err)=>{
    //에러 처리
});
```

```
//await 사용시
try{
    await conf.publish( [localMedia] );
} catch (err) {
    //에러 처리
}
```

### **미디어 구독 에러 처리**
미디어 구독을 위해 호출하는 subscribe API도 실패시 에러를 반환합니다.

```
//promise
conf.subscribe(비디오아이디배열).catch((err)=>{
    //에러 처리
});
```

```
//await 사용시
try{
    await this.conf.subscribe(비디오아이디배열);
} catch (err) {
    //에러 처리
}
```

### **P2P 연결 실패 에러**
ConnectLive는 미디어 서버와 클라이언트가 p2p로 연결되어 오디오와 비디오를 송수신 합니다. 이때 p2p로 연결된 미디어 서버와의 네트워크 상황 등에 의해 연결을 실패할 수 있습니다. 이러한 상황에 대응할 수 있도록 화상 회의 객체에 error 콜백을 등록할 수 있습니다. 해당 이벤트 안에서 disconnect를 호출하고 초기 화면으로 이동한다 등의 에러 처리가 필요합니다.

```
conf.on('error', ()=>{
    //에러 처리
    conf.disconnect();
});
```

## **예제 소개**
### **실행하기**
예제는 vue 프로젝트로 작성되었습니다. 구동하는 방법은 아래와 같으며 구동하면 localhost:8080 으로 실행됩니다.

```
$ yarn install

$ yarn serve
```

### **예제 설명**
l  Connectlive javascript sdk는 package.json에 명시되어 install 됩니다.

l  예제를 구현한 컴포넌트는 src/components/HelloWorld.vue 파일에 존재합니다.

l  remoteVideo.vue는 상대방의 비디오를 표현하는 컴포넌트입니다.

l  voiceMeter.vue는 오디오 레벨을 표현하는 컴포넌트입니다.

l  src/components/HelloWorld.vue 내 서비스아이디, 서비스키를 입력해야 합니다.
```
await ConnectLive.signIn({
    serviceId: '서비스아이디를 입력합니다',
    serviceKey: '서비스키를 입력합니다',
    secret: '시크릿키를 입력합니다'
});
```


### **화면 기능**

1. 연결하기 버튼을 클릭하면 명시된 룸아이디로 접속합니다.
2. 초기에는 연결하기 버튼이 노출되고 룸에 연결되면 연결해제하기 버튼이 노출됩니다.
3. 연결이 완료된 후에는 화면공유적용 버튼이 노출되고 화면 공유 중이라면 화면공유해제 버튼이 노출됩니다.
4. 카메라 목록을 가져와 select로 표현합니다.
5. 선택된 카메라로 변경할 수 있습니다.
6. 자신의 로컬 비디오가 표시됩니다.
7. 상대방의 리모트 비디오들이 표시됩니다. 좌측 상단의 노란색 표시는 현재 오디오를 점유하고 있는 참여자들에 대한 표시입니다.
8. 좌측 비디오를 클릭하면 품질을 향상시키면서 가운데 표시합니다. high 비디오는 하나만 유지되어 다른 비디오를 선택하면 이번 비디오는 이전 상태로 돌아가고 선택된 현재 비디오가 high로 설정됩니다.
