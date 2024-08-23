느낀점: AWS DeepRacer 플랫폼에서 제공하는 인터페이스를 통해, 다양한 매개변수를 조정하면서 모델을 훈련할 수 있었습니다. 

- 변경하는 매개변수
  
1.Learning Rate(학습률): 시간 VS 세밀한 학습 => "학습률이 크면 시간이 적게 걸리지만, 세밀한 학습이 되지 않는다.

2.Discount Factor(할인률): 즉각적인 보상 vs 미래에 대한 보상 => 할인율이 크면 미래에 대한 보상을 중요하게 여긴다.

3.Batch Size (배치 크기) : 메모리 사용량 vs 경사 계산의 변동성 => 배치가 크면 메모리 사용량이 높고 더 정확한 경사를 계산할 수 있다. (경사는, 손실함수가 최소가 되는 방향으로 가중치 set을 정하는것)

4.트랙 선택 (최대한 실제 경기장과 비슷한 트랙으로 훈련.)

5.시뮬레이션 속도=> 시뮬레이션 속도가 작을수록, 오래걸리지만 정확도가 높은 훈련임.

6.센서 설정 => 카메라, 스테레오 카메라, Lidar중 시뮬레이션 공간에서 어떤것을 사용할지

---

- AWS DeepRacer는 Amazon Web Services(AWS)에서 제공하는 자율주행 차량 플랫폼으로, 사용자가 강화 학습(Reinforcement Learning, RL)을 통해 자율주행 모델을 개발하고 훈련할 수 있게 해줍니다.

- DeepRacer 차량: AWS DeepRacer는 1/18 스케일의 소형 자율주행 차량입니다. 이 차량은 전방 카메라와 컴퓨팅 유닛(내부에는 Intel Atom 프로세서와 다양한 센서가 장착되어 있음)을 통해 자율적으로 경로를 학습하고 주행할 수 있습니다.

- 강화 학습 (Reinforcement Learning): DeepRacer는 강화 학습 알고리즘을 사용하여 차량이 트랙 위에서 주행하는 방법을 학습합니다. 강화 학습은 에이전트(여기서는 차량)가 주어진 환경에서 행동을 취하고, 그 결과에 따라 보상을 받으면서 최적의 행동을 학습하는 방법입니다.

- 모델 훈련: 사용자는 AWS Management Console에서 DeepRacer 환경을 설정하고, 다양한 트랙에서 시뮬레이션을 통해 모델을 훈련시킬 수 있습니다. 훈련된 모델은 실제 DeepRacer 차량에 배포하여 실세계에서 테스트할 수 있습니다.

- 가상 리그와 대회: AWS는 DeepRacer 리그라는 글로벌 대회를 운영하고 있으며, 사용자는 자신이 개발한 모델로 가상 리그에 참여하거나 오프라인 대회에 참가할 수 있습니다. 이를 통해 다른 사람들과 경쟁하고 자신의 모델을 평가할 수 있습니다.
***
- 보상 함수: DeepRacer의 성능을 결정하는 핵심 요소 중 하나는 보상 함수입니다. 보상 함수는 차량이 경로를 잘 따를수록 높은 보상을 주는 방식으로 설정됩니다. 사용자는 자신만의 보상 함수를 설계하여 차량이 최적의 주행을 하도록 유도할 수 있습니다.
---

AWS 강화학습 DeepRacer 플랫폼을 이용한 자율주행

******AWS 플랫폼내에서 셋팅하기
카메라 , 스테레오 카메라 , 라이더 센서 3개중 선택
우리는 카메랑랑 라이더센서 있는 거 선택
front-facing camera 앞에 사물 1step당 1초에 15번 사진
스테레오 카메라  사물 인식까지
라이더 센서 뒤 쪽 사물 인식

보상함수 초점 
코너를 돌 때 왼쪽으로만 가게끔 보상함수를 높였음
evaluaton 을 보면 실제로 차가 트택 중앙 기준 왼 쪽으로 가는게 보임Speed: [ 0.65 : 1.9 ] m/s
Steering angle: [ -30 : 30 ] °
하이퍼 파라미터 중
Learning rate 는 0.0003 이 기본값이지만 0.0001로 설정
Discount factor	0.999 기본 0.9로 설정
나머지는 다 기본값
progress 0~100% 100% 가 되면
**********************************************************************************
import math

def reward_function(params):

waypoints = params['waypoints']
closest_waypoints = params['closest_waypoints']
heading = params['heading']
distance_from_center = params['distance_from_center']
track_width = params['track_width']
progress = params['progress']
steps = params['steps']

reward = 1.0- (distance_from_center/(track_width/2))**(1/4) + progress/steps

next_point = waypoints[closest_waypoints[1]]
prev_point = waypoints[closest_waypoints[0]]

track_direction = math.atan2(next_point[1] - prev_point[1], next_point[0] - prev_point[0])

track_direction = math.degrees(track_direction)

direction_diff = abs(track_direction - heading)
if direction_diff > 180:
direction_diff = 360 - direction_diff

DIRECTION_THRESHOLD = 10.0 #차량이 트랙 방향과 10도 이상 차이가 나면 보상이 절반으로 감소한다. => 차량이 트랙 방향을 잘 따라가도록 유도.
if direction_diff > DIRECTION_THRESHOLD:
reward *= 0.5

return float(reward)
**********************************************************************************************
각 파라미터에 대한 설명
params: AWS DeepRacer 플랫폼에서 주행 상황에 대한 다양한 정보를 포함하는 딕셔너리로, 이 함수에서 사용되는 주요 파라미터는 다음과 같습니다

waypoints: 트랙의 각 점(웨이포인트)의 좌표 리스트. 이 점들은 차량이 따라가야 하는 경로를 나타냅니다.

closest_waypoints: 차량이 현재 위치한 트랙의 두 웨이포인트를 인덱스로 나타낸 배열. closest_waypoints[0]은 이전 웨이포인트, closest_waypoints[1]은 다음 웨이포인트를 나타냅니다.

heading: 차량의 현재 진행 방향을 나타내는 각도(도 단위).

distance_from_center: 트랙의 중앙선으로부터 차량의 현재 위치까지의 거리.

track_width: 트랙의 전체 너비.

progress: 차량이 전체 트랙에서 얼마나 진행했는지를 백분율로 나타낸 값 (0에서 100%).

steps: 현재까지 차량이 주행한 스텝 수.

***우승 POINT****************

 1. **(1/X) 부분에서 X를 4로 했을때 제일 좋은결과가 나왔음.
 reward = 1.0- (distance_from_center/(track_width/2))**(1/4) + progress/steps
'(distance_from_center/(track_width/2))**(1/4)' 부분은 -> 차량이 트랙의 중앙에 가까울 수록 값이 작아져서 높은 보상을 받게 합니다.
'progress/ steps ' 는 차량이 트랙을 얼마나 효율적으로 따라가고 있는지를 반영합니다 -> 더 많은 진행(progress)를 적은 스텝으로 달성할수록 높은 보상을 제공합니다.
2. 적절한 DIRECTION_THRESOLD 조절. 
