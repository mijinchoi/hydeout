---
layout: post
title: Introducing Code
---

import time
import smbus
import RPi.GPIO as GPIO
import os
import vlc

 
GPIO.setmode(GPIO.BCM)
 
GPIO.setup(14, GPIO.IN)
 
GPIO.setup(15, GPIO.IN)
 
GPIO.setup(18, GPIO.IN)
 
GPIO.setup(23, GPIO.IN)

GPIO.setup(27, GPIO.IN)
 
GPIO.setup(22, GPIO.IN)

GPIO.setup(26, GPIO.IN)



I2C_ADDR = 0x27  # I2C device address
LCD_WIDTH = 16  # Maximum characters per line

# Define some device constants
LCD_CHR = 1  # Mode - Sending data
LCD_CMD = 0  # Mode - Sending command

LCD_LINE_1 = 0x80  # LCD RAM address for the 1st line
LCD_LINE_2 = 0xC0  # LCD RAM address for the 2nd line
LCD_LINE_3 = 0x94  # LCD RAM address for the 3rd line
LCD_LINE_4 = 0xD4  # LCD RAM address for the 4th line


LCD_BACKLIGHT = 0x08  # On
# LCD_BACKLIGHT = 0x00  # Off

ENABLE = 0b00000100  # Enable bit

# Timing constants
E_PULSE = 0.0005
E_DELAY = 0.0005

# Open I2C interface
# bus = smbus.SMBus(0)  # Rev 1 Pi uses 0
bus = smbus.SMBus(1)  # Rev 2 Pi uses 1


def lcd_init():
    # Initialise display
    lcd_byte(0x33, LCD_CMD)  # 110011 Initialise
    lcd_byte(0x32, LCD_CMD)  # 110010 Initialise
    lcd_byte(0x06, LCD_CMD)  # 000110 Cursor move direction
    lcd_byte(0x0C, LCD_CMD)  # 001100 Display On,Cursor Off, Blink Off
    lcd_byte(0x28, LCD_CMD)  # 101000 Data length, number of lines, font size
    lcd_byte(0x01, LCD_CMD)  # 000001 Clear display
    time.sleep(E_DELAY)


def lcd_byte(bits, mode):
    # Send byte to data pins
    # bits = the data
    # mode = 1 for data
    #        0 for command

    bits_high = mode | (bits & 0xF0) | LCD_BACKLIGHT
    bits_low = mode | ((bits << 4) & 0xF0) | LCD_BACKLIGHT

    # High bits
    bus.write_byte(I2C_ADDR, bits_high)
    lcd_toggle_enable(bits_high)

    # Low bits
    bus.write_byte(I2C_ADDR, bits_low)
    lcd_toggle_enable(bits_low)


def lcd_toggle_enable(bits):
    # Toggle enable
    time.sleep(E_DELAY)
    bus.write_byte(I2C_ADDR, (bits | ENABLE))
    time.sleep(E_PULSE)
    bus.write_byte(I2C_ADDR, (bits & ~ENABLE))
    time.sleep(E_DELAY)


def lcd_string(message, line):
    # Send string to display

    message = message.ljust(LCD_WIDTH, " ")

    lcd_byte(line, LCD_CMD)

    for x in range(LCD_WIDTH):
        lcd_byte(ord(message[x]), LCD_CHR)


def main():
    # Main program block
    # Initialise display
    lcd_init()


if __name__ == '__main__':

    try:
        main()
    except KeyboardInterrupt:
        pass
    finally:
        lcd_byte(0x01, LCD_CMD)

        i = 0


instance = vlc.Instance()
 
player = instance.media_player_new()
 
path = "/home/pi/Downloads/"
 
a = os.listdir(path)
 
file = ("/home/pi/Downloads/" + a[i])

print(a[i])
 
media = instance.media_new(file)
 
player.set_media(media)
 
time.sleep(1)
  
n = 0
 
num = player.get_state()

b = ['3:44 -IU','3:38 - IU','3:43 - IU','5:37 - IU','3:53 - IU','4:26 - IU']

v=0

vol = 50

player.audio_set_volume(100)
    

while True:
    num = player.get_state()

    if (num == 3 or num == 4):

        lcd_string(    str(a[i]) + str("")    , LCD_LINE_1)
        lcd_string(    str(b[v]) + str("")    , LCD_LINE_2)
     
    if GPIO.input(14) == 0:
        #num = player.get_state()
     
        n = n + 1
     
        if n == 1: # first click music play

            
     
            player.play()  # play a[0] possible??

            print(i)

            print('music start')
     
            time.sleep(0.5)
     
        else:#after, play and pause
     
            player.pause()
     
            time.sleep(0.5)
     
    if GPIO.input(15) == 0: #next music play #time sleep problem?
            #num = player.get_state()

        i = i + 1

        v = v + 1
     
        if (i==len(a)): #if now play last > back to a[0]
     
            player.stop()
     
            i = 0

            v = 0
                
            print(i)

            print('fisrt music')
     
            file = ("/home/pi/Downloads/" + a[i])
            media = instance.media_new(file)
            player.set_media(media)
     
            player.play()
     
            time.sleep(0.5)
     


        else:

            print(i)
      
            player.stop()

                #new i value receive?
            file = ("/home/pi/Downloads/" + a[i])
            media = instance.media_new(file)
            player.set_media(media)
     
     
            player.play()

            lcd_string(    str(a[i])     , LCD_LINE_1)


            print('next music')
     
            time.sleep(0.5)
     
     
    if GPIO.input(18) == 0: #previous music
        #num = player.get_state()

        i = i - 1

        v = v - 1
     
        if (i <= -1): #if now play a[0] > go to last music
                
            player.stop()

            i = len(a)

            v = 6

            print(i)

            print('last music')
     
            file = ("/home/pi/Downloads/" + a[i])
            media = instance.media_new(file)
            player.set_media(media)
            player.play()

            time.sleep(0.5)

        else:
            player.stop()

            print(i)

            print('previous music')
     
            file = ("/home/pi/Downloads/" + a[i])
            media = instance.media_new(file)
            player.set_media(media)
            player.play()

            time.sleep(0.5)
    
    if GPIO.input(27) == 0:   #volume down

        vol = vol - 10

        player.audio_set_volume(vol)

        print("down")
        
    if GPIO.input(22) == 0:    #volume up
        vol = vol + 10

        player.audio_set_volume(vol)        

        print("up")

    if GPIO.input(23) == 0: #program stop
        num = player.get_state()
     
        if(num==3 or num ==4):
     
            player.stop()

            lcd_string(     str("")    , LCD_LINE_1)
            lcd_string(     str("")    , LCD_LINE_2)
            
            time.sleep(0.5)

            break

    if GPIO.input(26) == 0:
        player.stop()
        i = 0
        instance = vlc.Instance()
        player = instance.media_player_new()
        path = "/home/pi/Music/"
        a = os.listdir(path)
        file = ("/home/pi/Music/" + a[i])
        media = instance.media_new(file)
        player.set_media(media)
        time.sleep(1)
        
        n = 0
    
        num = player.get_state()
        c = ['5.:12 - Piano', '5:00 - Piano', '3:52 - Piano']

        v = 0
        

        lcd_string(    str("List change")   , LCD_LINE_1)
        lcd_string(    str("")    , LCD_LINE_2)
        

        while True:
            num = player.get_state()

            if (num == 3 or num == 4):
 
                lcd_string(    str(a[i]) + str("")    , LCD_LINE_1)
                lcd_string(    str(c[v]) + str("")    , LCD_LINE_2)
            if GPIO.input(14) == 0:
                if n == 0:# first click music play
                    n = n + 1
                         

                    print(i)
     
                    print('music start')

                    player.play()
                    time.sleep(0.5)
                        
                else:#after, play and pause
                    player.pause()
                    time.sleep(0.5)
     
     
            if GPIO.input(15) == 0: #next music play #time sleep problem?
                #num = player.get_state()
                i = i + 1
     
                v = v + 1
                if (i==len(a)): #if now play last > back to a[0]
         
                    player.stop()
         
                    i = 0
     
                    v = 0
                    
                    print(i)
     
                    print('fisrt music')
         
                    file = ("/home/pi/Music/" + a[i])
                    media = instance.media_new(file)
                    player.set_media(media)         
                    player.play()
         
                    time.sleep(0.5)
     
                else:
     
                    print(i)
          
                    player.stop()#new i value receive?
                    
                    file = ("/home/pi/Music/" + a[i])
                    media = instance.media_new(file)
                    player.set_media(media)
         
         
                    player.play()
    
                    lcd_string(    str(a[i])     , LCD_LINE_1)     
     
                    print('next music')
         
                    time.sleep(0.5)
         
         
            if GPIO.input(18) == 0: #previous music
                    
                    #num = player.get_state()
                i = i - 1
     
                v = v - 1
     
                if (i <= -1): #if now play a[0] > go to last music
                    
                    player.stop()
     
                    i = 5
     
                    v = 5
     
                    print(i)
     
                    print('last music')
         
                    file = ("/home/pi/Music/" + a[i])
                    media = instance.media_new(file)
                    player.set_media(media)
                    player.play()
     
                    time.sleep(0.5)

                else:
                    player.stop()
                             
                    print(i)
                    print('previous music')
         
                    file = ("/home/pi/Music/" + a[i])
                    media = instance.media_new(file)
                    player.set_media(media)
                    player.play()
     
                    time.sleep(0.5)
                    
            if GPIO.input(27) == 0:   #volume down

                vol = vol - 10

                player.audio_set_volume(vol)

                print("down")
                        
            if GPIO.input(22) == 0:    #volume up
                vol = vol + 10

                player.audio_set_volume(vol)        

                print("up")

         
            if GPIO.input(23) == 0: #program stop
                num = player.get_state()

                if(num==3 or num ==4):
                    player.stop()
     
                    lcd_string(     str("")    , LCD_LINE_1)
                    lcd_string(     str("")    , LCD_LINE_2)
                
                    time.sleep(0.5)     
                    break
