```
@startuml
skinparam componentStyle rectangle
skinparam shadowing false

package "VL53L5CX Driver" {
  component "Platform\n(platform.h/c)" as plat
  component "Buffers\n(vl53l5cx_buffers.h)" as buf
  component "API Core\n(vl53l5cx_api.h/c)" as api
  component "Detection Thresholds Plugin\n(vl53l5cx_plugin_detection_thresholds.h/c)" as dt
  component "Motion Indicator Plugin\n(vl53l5cx_plugin_motion_indicator.h/c)" as mi
  component "XTalk Plugin\n(vl53l5cx_plugin_xtalk.h/c)" as xt

  api --> plat : depends on (I2C comms)
  api --> buf : uses (firmware data)
  dt --> api : extends (threshold config)
  mi --> api : extends (motion detection)
  xt --> api : extends (xtalk calibration)
}

component "Example Application\n(example.c)" as ex
ex --> api : uses (demo ranging)

@enduml
```

<img width="1114" height="376" alt="image" src="https://github.com/user-attachments/assets/94081f7d-5b23-4d20-9a6d-f23068156351" />


```
@startuml
skinparam shadowing false
actor User
participant "Main (example.c)" as main
participant "VL53L5CX API" as api
participant "Platform" as plat
participant "Sensor (Hardware)" as sensor

User -> main: run main()
main -> plat: VL53L5CX_Comms_Init("/dev/i2c-1")
plat --> main: status

main -> api: vl53l5cx_is_alive()
api -> plat: RdByte (check GO2 status)
plat -> sensor: I2C read
sensor --> plat: data
plat --> api: status
api --> main: isAlive

main -> api: vl53l5cx_init() (load firmware, config)
api -> plat: multiple WrMulti/RdMulti (DCI ops)
plat -> sensor: I2C write/read
sensor --> plat: responses
plat --> api: status
api --> main: status

main -> api: vl53l5cx_start_ranging()
api -> plat: WrMulti (start command)
plat -> sensor: I2C write
api --> main: status

loop 10 times
  main -> api: vl53l5cx_check_data_ready()
  api -> plat: RdMulti (UI_CMD_STATUS)
  plat -> sensor: I2C read
  sensor --> plat: data ready?
  plat --> api: isReady
  api --> main: isReady

  alt if isReady
    main -> api: vl53l5cx_get_ranging_data()
    api -> plat: RdMulti (results data)
    plat -> sensor: I2C read
    sensor --> plat: ResultsData
    plat --> api: data
    api --> main: Results (print distances)
  end
  note right of main: WaitMs(5)
end

main -> api: vl53l5cx_stop_ranging()
api -> plat: WrMulti (stop command)
plat -> sensor: I2C write
api --> main: status

main -> plat: VL53L5CX_Comms_Close()
plat --> main: status

@enduml
```

<img width="854" height="1356" alt="image" src="https://github.com/user-attachments/assets/5ceefa4b-57f8-417d-a85f-691dfb8816e2" />

