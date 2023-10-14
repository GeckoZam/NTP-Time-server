# Hora de internet (NTP Time server)

## Código

```python
"""
Alumno: Salazar Diaz Samuel
No. Control: 19211729
Materia: Sistemas Programables
"""
import framebuf, sys
import utime
import network
import time
from machine import Pin, I2C
from ssd1306 import SSD1306_I2C
import time_images

pix_res_x = 128
pix_res_y = 64

def init_i2c(scl_pin, sda_pin):
  # Inicializamos la pantalla OLED
  i2c_dev = I2C(1, scl=Pin(scl_pin), sda=Pin(sda_pin), freq=200000)
  i2c_addr = [hex(ii) for ii in i2c_dev.scan()]
  
  if not i2c_addr:
    print('No I2C Display Found')
    sys.exit()  
  else:
    print("I2C Address      : {}".format(i2c_addr[0]))
    print("I2C Configuration: {}".format(i2c_dev))
  
  return i2c_dev

# Se le da formato de 12 hr al reloj que mostrará
def time_format(hour, minutes):
  f_hour = hour % 12 if hour != 12 else hour
  f_hour = ("0" + str(f_hour)) if f_hour < 10 else f_hour
  f_minutes = ("0" + str(minutes)) if minutes < 10 else minutes
  meridiem = "AM" if hour < 12 else "PM"
  return f"{f_hour}:{f_minutes} {meridiem}"

# Se asigna las imagenes dependiendo de la hora que se obtuvo
def match_image_time(hour):
  if 6 <= hour < 12:
    image = time_images.MORNING
  elif 12 <= hour < 20:
    image = time_images.EVENING
  elif (20 <= hour <= 24) or (0 <= hour < 6):
    image = time_images.NIGHT
  else:
    raise ValueError("Hour off limits")
  return framebuf.FrameBuffer(image, 30, 30, framebuf.MONO_HLSB)

# Muestra la hora
def display_time(oled):
  last_hour = -1
  x_center = (pix_res_x - 30) // 2
  hour, minutes = 0, 0
# Muestra la region actual
  while True:
    oled.text("Horario de", 5, 0)
    oled.text("America/Tijuana", 5, 10)
    oled.show()
    hour, minutes = time.localtime()[3:5]
    hour += 1
    oled.text(time_format(hour, minutes), 5, 20)

    if hour != last_hour:
      image = match_image_time(hour)
      oled.blit(image, x_center, 30)
   
    oled.show()
    time.sleep(1)
    oled.fill(0)

# Se llama a la funcion para conectarse a internet
def main():
  internet.connection()
  internet.set_time()

  i2c_dev = init_i2c(scl_pin=27, sda_pin=26)
  oled = SSD1306_I2C(pix_res_x, pix_res_y, i2c_dev)
  display_time(oled)

if __name__ == '__main__':
  main()
```

# Imagenes de proyecto funcionando

