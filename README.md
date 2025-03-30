# CircuitPython-DMX
## Using Circuitpython to Move Motors, Change Colors and Make sound with DMX Interface

I am studying ways to create stage devices with movement, light and sound that can be controlled from the operating booth in a theatrical show.

This first prototype was developed using the CytronTech Maker Pi RP2040 board, with CircuitPython and the CircuitPython DMX Receiver library https://github.com/mydana/CircuitPython_DMX_Receiver

The system receives the DMX signal, breaks down each of the channels and assigns the values ​​to specific devices on the board, activating Neopixel LEDs with specific colors, moving servomotors and motors. It would also be possible to assign sounds and have the board's buzzer emit them.

```
import board, neopixel
from simpleio import map_range
import pwmio
from adafruit_motor import servo, motor

# Copy files from https://github.com/mydana/CircuitPython_DMX_Receiver onto lib folder
from dmx_receiver import DmxReceiver 

DMX_PIN = board.GP1 # Módulo RS 485
PIXEL_PIN = board.GP18 # neopixel on board, dois LEDS RGB

# Read 16 sequential channels from DMX starting at Slot 0
dmx_receiver = DmxReceiver(pin=DMX_PIN, slot=0) 
       # pin: The pin to retrieve data from.
       # dmx_basis: If True start slot numbers at 1 not 0. - Use true, to follow DMX Channel standards
       # slot: The first DMX slot of 16 to listen for.

# Neopixel Setup
pixels = neopixel.NeoPixel(PIXEL_PIN, 2)

servo_pwm1 = pwmio.PWMOut(board.GP12, duty_cycle=2 ** 15, frequency=50)
servo_pwm2 = pwmio.PWMOut(board.GP13, duty_cycle=2 ** 15, frequency=50)
servo_pwm3 = pwmio.PWMOut(board.GP14, duty_cycle=2 ** 15, frequency=50)
servo_pwm4 = pwmio.PWMOut(board.GP15, duty_cycle=2 ** 15, frequency=50)

motor1_pwm_a = pwmio.PWMOut(board.GP9, frequency=50)
motor1_pwm_b = pwmio.PWMOut(board.GP8, frequency=50)
motor1 = motor.DCMotor(motor1_pwm_a, motor1_pwm_b)

motor2_pwm_a = pwmio.PWMOut(board.GP10, frequency=50)
motor2_pwm_b = pwmio.PWMOut(board.GP11, frequency=50)
motor2 = motor.DCMotor(motor2_pwm_a, motor2_pwm_b)

servo1 = servo.Servo(servo_pwm1)
servo2 = servo.Servo(servo_pwm2)
servo3 = servo.Servo(servo_pwm3)
servo4 = servo.Servo(servo_pwm4)


for data in dmx_receiver:
    if data is not None:
        # Convert Bytes to a Integer List - Debug only
        dmx_list = list(data)
        print(dmx_list)
        
        # CHANNEL 1, 2, 3 e 4 - RGB and Brightness
        color_red = dmx_list[0] # Channel 1
        color_green = dmx_list[1] # Channel 2
        color_blue = dmx_list[2] # Channel 3
        pixels.brightness = map_range (dmx_list[3], 0,255, 0.0,1.0) # Channel 4
        
        pixels[0] = (color_red, color_green, color_blue)
        pixels[1] = (color_red, color_green, color_blue)        
        pixels.show()
       
        # Channels 5 to 8, Four Servo positions
        servo1.angle = map_range (dmx_list[4], 0,255, 0,180) # Channel 5
        servo2.angle = map_range (dmx_list[5], 0,255, 0,180) # Channel 6
        servo3.angle = map_range (dmx_list[6], 0,255, 0,180) # Channel 7
        servo4.angle = map_range (dmx_list[7], 0,255, 0,180) # Channel 8
        
        # Channels 9 and 10 - Motor 1 Speed and Direction
        motor1_speed = map_range(dmx_list[8], 0,255,0, 1.0) # Channel 9
        motor1_direction = dmx_list[9] # Channel 10
        
        if motor1_speed == 0:
            motor1.throttle = None # Shutdown Motor
        else:
            if motor1_direction in range (0,127):
                motor1.throttle = motor1_speed
            else:
                motor1.throttle = motor1_speed * -1.0

        # Channels 11 and 12 - Motor 2 Speed and Direction  
        motor2_speed =  map_range(dmx_list[10], 0,255,0, 1.0) # Channel 11
        motor2_direction = dmx_list[11] # Channel 12

        if motor2_speed == 0:
            motor2.throttle = None
        else:
            if motor2_direction in range (0,127):
                motor2.throttle = motor2_speed
            else:
                motor2.throttle = motor2_speed * -1.0


```
