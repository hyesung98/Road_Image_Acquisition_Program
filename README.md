

Road Image Acquistion Program
===============

Overview
--------

Based on ROS melodic and QT C++

- Camera : Basler ace acA4112-8gc 
- GNSS :  novatel PWRPAK7

--------

### Parameter Setting

1. **novatel_gps_driver**

   Link : <https://github.com/swri-robotics/novatel_gps_driver>

   원하는 Topic을 받기 위해 내부 launch 파일 수정

   - novatel_gps_driver/launch/tester_for_eth.launch 설정

   ```java
   <?xml version="1.0"?>
   <launch>
     <node name="novatel"
           pkg="nodelet" type="nodelet"
           args="standalone novatel_gps_driver/novatel_gps_nodelet">
       <rosparam>
         verbose: true
         connection_type: tcp
         device: 192.168.1.210:3005
         imu_sample_rate: -1
         polling_period: 0.2
         use_binary_messages: true
         publish_novatel_positions: true
         publish_novatel_utm_positions: true
         publish_imu_messages: true
         imu_frame_id: /imu
         publish_nmea_messages: true
         frame_id: /gps
       </rosparam>
     </node>
   </launch>
   ```

2. **basler_ros_driver**

     카메라수와  serial number를 입력해준다.

   - basler_ros_driver/config/default.yaml 설정 (예시: 아래)

   ```yaml
   camera_count : 4
   
   camera1:
       name: cam1
       serial_number: 40097766
   
   camera2:
       name: cam2
       serial_number: 40097767
   
   camera3:
       name: cam3
       serial_number: 40097977
   
   camera4:
       name: cam4
       serial_number: 40097973
   ```

2. **road_recorder**

     카메라 갯수와 mode를 입력해준다.

   - road_recorder/config/default.yaml 설정 (예시: 아래)

   ```yaml
   # camera mode false --> simulation / true --> real world
   
   viewer_mode: false
   camera_count: 4
   ```
   
      - `viewer_mode` : 모드 설정 
   
   | value |      mode       |
   | :---: | :-------------: |
   | true  |    촬영 모드    |
   | false | 시뮬레이션 모드 |
   
     - `camera_count` : 카메라수 설정


--------

### .launch 실행

**road_recorder**  실행

```bash
roslaunch road_rocorder run.launch
```

**novaltel_gps_driver**, **basler_ros_driver** 실행

```bash
roslaunch road_recorder sensor.launch
```

--------

### Topic & Service 구성

![Alt text](/image/topic_diagram.png)

|           Node Name           |      Topic & Service      |                         Description                          |
| :---------------------------: | :-----------------------: | :----------------------------------------------------------: |
|       basler-ros-driver       |    Camera info (Topic)    | 받은 Trigger Service 갯수와 저장에 성공한 이미지 갯수를 담고 있는 Topic |
|       basler-ros-driver       |     Raw Image (Topic)     |       Trigger Service를 받으면 보내는 Image type Topic       |
| road_image_acquistion_program |    Camera info (Topic)    | basler-ros-driver에 사진촬영을 요청하는 Service Service를 보내면 Raw  Image를 받을 수 있다. |
| road_image_acquistion_program | Image Directory (Service) |         촬영된 .bmp 의 저장 경로를 지정하는 Service          |
| road_image_acquistion_program |  Bag Directory (Service)  |         촬영된 .bmp 의 저장 경로를 지정하는 Service          |
| road_image_acquistion_program |  Start & Stop (Service)   |    bag recorder의 촬영을 시작, 중지 시킬 수 있는 Service     |
|      novatel-gps-driver       |      bestpos (Topic)      |     Latitude, Longitude, Accuracy 값을  담고 있는 Topic      |
|      novatel-gps-driver       |      bestutm (Topic)      |           UTM 좌표상의 x, y, z값을 담고 있는 Topic           |
|      novatel-gps-driver       |      Inspva (Topic)       |            Roll, Pitch, Yaw 값을 담고 있는 Topic             |

--------

  ### GUI 구성

 ![Alt text](/image/UI_configuration.png)

 - **Map View**

   KaKao Map API를 사용한 지도 표출 및 interpolation된 좌표와 현재 위치를 지도에 표시함

 - **Camera View**

   Trigger 서비스를 보내고 받은 이미지를 뛰움

 - **Fucntion View**

   카메라의 녹화 및 정지 및 파일 저장 위치를 설정할 수 있음. 촬영조건(거리, 시간)과 그 값을 선택할 수 있음  

    - **Record**

      - record 버튼을 누르면 녹화가 시작되고 다시 누르면 중지

      - shot버튼을 누르면 누른 시점을 촬영

      - 선택 버튼 그룹을 통해 시간 또는 거리기준으로 촬영 모드 선택 

        |   Mode   |       Discription       |
        | :------: | :---------------------: |
        |   Time   | 설정 시간 기준으로 촬영 |
        | Distance | 설정 거리 기준으로 촬영 |

      ![Alt text](/image/record_view.png)

    - **Directory** 

      - 저장될 폴더(bag, csv, image)의 이름과 경로를 지정

      ![Alt text](/image/file_view.png)

 - **GNSS View**

   GNSS PWRPAK7을 통해 받은 위경도 데이터와 status를 표시 
