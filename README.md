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
params: AWS DeepRacer 플랫폼에서 주행 상황에 대한 다양한 정보를 포함하는 딕셔너리로, 이 함수에서 사용되는 주요 파라미터는 다음과 같습니다:
waypoints: 트랙의 각 점(웨이포인트)의 좌표 리스트. 이 점들은 차량이 따라가야 하는 경로를 나타냅니다.
closest_waypoints: 차량이 현재 위치한 트랙의 두 웨이포인트를 인덱스로 나타낸 배열. closest_waypoints[0]은 이전 웨이포인트, closest_waypoints[1]은 다음 웨이포인트를 나타냅니다.
heading: 차량의 현재 진행 방향을 나타내는 각도(도 단위).
distance_from_center: 트랙의 중앙선으로부터 차량의 현재 위치까지의 거리.
track_width: 트랙의 전체 너비.
progress: 차량이 전체 트랙에서 얼마나 진행했는지를 백분율로 나타낸 값 (0에서 100%).
steps: 현재까지 차량이 주행한 스텝 수.
***우승 POINT

 1. **(1/X) 부분에서 X를 4로 했을때 제일 좋은결과가 나왔음.
 reward = 1.0- (distance_from_center/(track_width/2))**(1/4) + progress/steps
'(distance_from_center/(track_width/2))**(1/4)' 부분은 -> 차량이 트랙의 중앙에 가까울 수록 값이 작아져서 높은 보상을 받게 합니다.
'progress/ steps ' 는 차량이 트랙을 얼마나 효율적으로 따라가고 있는지를 반영합니다 -> 더 많은 진행(progress)를 적은 스텝으로 달성할수록 높은 보상을 제공합니다.
3. 적절한 DIRECTION_THRESOLD 조절. 
