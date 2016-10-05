## CTF.COM.UA получение флага Suspicious AVI

За получения флага для данного задания, дается 450 поинтов и следующее условие

`RU: Эксперт в области Информационной безопасности начал создавать уроки для киберармии, но он по иронии судьбы оказался шпионом! Вам приказали проверить один из его видеоуроков на предмет передачи секретной информации.`

 ![intro2](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/intro2.png)

И так, нам предстоит исследовать avi файл. Для этого скачаем его и попробуем посмотреть что же там такого есть, что можно увидеть невооруженным взглядом и запускаем.

 ![avi](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/avi.png)

Видим забавный ролик, где свинюшки пытаются что то сделать с ПК, но сразу же заметно, что звук отстает от изображения вполне прилично. Скорее всего что то в звуковой дорожке, но для начала нужно понять что же есть в скрытом содержимом AVI файла. AVI файл представляет из себя последовательность изображений.

Зная то что стенография не применима к AVI как таковому, но применима к файлам изображений, разложим наш ролик на составляющие картинки. Для этоо воспользуемся командой `ffmpeg -i avi.avi -f image2 image-%03d.png`

Этой командой мы получаем большое кол-во PNG файлов пронумерованных по кол-ву кадров.

![VirtualBox_Kali2016.2_25_09_2016_15_59_31](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_25_09_2016_15_59_31.png)

При выполнении данной команды также в глаза бросается ссылка из метаданных AVI файла, которая как бы намекает на то, что по этому адресу содержится некая копия. При помощи этой копии мы возможно сможем сравнить файлы покадрово и получить информацию. Идем и качаем данный файл по ссылке и запускаем его чтобы посмотреть что же там.

 ![hehehe](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/hehehe.png)

Видим туже самую картину, но что забавно в этой версии звук и картинка впорядке. Значит предполагаем что это исходный файл. Также проделываем над этим файлом операцию по извлечению кадров. Теперь у нас имеются две папки с изображениями кадров.

Теперь по всем правилам жанра, нам нужно сравнить каждый кадр с каждым из двух файлов попарно. Для этого сущесвует много способов, но в моем случае binwally очень неплохо справляется.

Выполняем команду `python ~/tools/binwally/binwally.py avi_avi/ hehehe_avi/` и тут же видим результат:

![VirtualBox_Kali2016.2_04_10_2016_21_17_31](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_04_10_2016_21_17_31.png)

И прокрутив ниже вижим что кадр под номером 174 отличается.

![VirtualBox_Kali2016.2_04_10_2016_21_17_58](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_04_10_2016_21_17_58.png)

Ну и конечно же на ум приходит сразу мысль, используя инструменты для детекта стего попробовать найти что нибудь интересное. Исползуя `ztego` пробуем посмотреть что можно увидеть.

![VirtualBox_Kali2016.2_25_09_2016_22_17_11](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_25_09_2016_22_17_11.png)

Но собствено говоря ничего интересного не получается обнаружить. Так что в дело вступает брутфорс всех возможных способов. Предположили что можно сравнить две картинки и выявить отличающиеся пиксели и при помощи ImageMagiс инструментов получилаьс вот такая каринка.

 ![photo_2016-10-04_21-25-14](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/photo_2016-10-04_21-25-14.jpg)

В верхней части изображения видна светлая полоска из точек - это как раз отличающиеся пиксели. Но при дальнейшем размышлении ошибочно предположили, что нужно использовать, только отличающиеся RGB данные пикселей, в итоге ничего внятного из информации не получалось.

Тогда после некоторого времени раздумия, пришла в голову слудующая идея - нужно выяснить насколько отличаются значения RGB.

В итоге написал небольшой скриптик который сравнивал значения RGB, каждого пикселя и в такой же последовательности выводить в строку.

В итоге получается вот такой вывод:

![VirtualBox_Kali2016.2_04_10_2016_20_59_59](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_04_10_2016_20_59_59.png)

Вот текстовое представление разниц между пикселями (т.е. для каждого пикселя тут три значения разница R, разница G и разница B, таким  образом для первого пикселя получается 0 0 1 - т.е. разница в B значении (R и G не отличаются), ну и так далее по аналогии):

``` 001010110101101100101101001011010010110100101101001011010011111000101011001010110010101100111100010111010011111000101011001011100101101100101101001011010011111000101011001111000101110100111110001011100010110100101101010110110010110100111110001010110010101100111100010111010011111000101101001011100010101100101011001010110010101100101011001010110010101100101011001011100010110101011011001011010010110100111110001010110011110001011101001111100010110100101101001011010010110100101110010110110010110100101101001011010011111000101011001010110011110001011101001111100010110100101101001011100010101100101011001010110010101100101011001010110010101100101110001011010101101100101101001111100010101100101011001010110010101100101011001111000101110100111110001011100101101100101101001011010010110100111110001010110011110001011101001111100010110100101101001011010010110100101110010110110010110100101101001111100010101100111100010111010011111000101101001011010010110100101101001011010010111000101101001011010010110100101110001011010101101100101101001011010010110100101101001011010011111000101011001111000101110100111110001011010010110100101110001011010010110100101101001011010010110100101101001011010010110100101101001011010010110100101101001011010010110100101101001011100010101100101011001010110010101100101011001010110010101100101011001010110010101100101011001010110010101100101011001011100010101100101011001010110010111001011011001011010010110100111110001010110011110001011101001111100010110100101101001011010010110100101110001011010010110100101101001011010101101100101101001111100010101100101011001111000101110100111110001011010010111000101011001010110010101100101011001010110010101100101011001010110010101100101110010110110010110100101101001111100010101100111100010111010011111000101110001011010010110101011011001011010011111000101011001010110011110001011101001111100010110100101110001010110010101100101011001010110010101100101011001010110010101100101110001011010101101100101101001011010011111000101011001111000101110100111110001011010010110100101110001011010101101100101101001111100010101100101011001111000101110100111110001011100011111000101101001011010101101100101101001011010011111000101011001010110010101100111100010111010011111000101110000```

Код скрипта для получения разницы:

```python
from PIL import Image
import random
import string

img = Image.open('image-174.png')
img2 = Image.open('image-174_hehehe.png')
img_pix = img.convert('RGB')
img_pix2 = img2.convert('RGB')

width, height = img.size
width2, height2 = img2.size

print("Image 1: ", width, height)
print("Image 2: ", width, height)

binstr = ""

for h in range(height):
  for w in range(width):
    r1, g1, b1 = img_pix.getpixel((w,h))
    r2, g2, b2 = img_pix2.getpixel((w,h))
    binstr += str(abs(r1-r2))
    binstr += str(abs(g1-g2))
    binstr += str(abs(b1-b2))

img.close()
img2.close()

print binstr
```

Поглядев на строки с нулями и единицами, решил что нужно попытать счастье и попробовать преобразовать эту вереницу 1 и 0 в текст при помощи одно из онлайн сервисов и вот что получилось:

![bin_to_brain_fuck](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/bin_to_brain_fuck.png)

Видим что перед нами ни что иное как `brainfuck` код. Для того чтобы проверить данное утверждение, пробуем также воспользоваться также онлайн сервисом и вот что получается.

![brainfuck_to_flag](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/brainfuck_to_flag.png)

И радости нет предела! Видим флаг и конечно же получаем +450 поинтов!

brainfuck код:

`+[----->+++<]>+.[-->+<]>.--[->++<]>-.++++++++.-[-->+<]>----.[--->++<]>--.+++++++.-[->+++++<]>.[--->+<]>----.[-->+<]>-----.---.-[----->+<]>--.---------------.++++++++++++++.+++.[-->+<]>----.----[->++<]>-.+++++++++.[-->+<]>.--[->++<]>-.++++++++.-[-->+<]>--.-[->++<]>.>--[-->+++<]>.`

Это конечно один из способов решения для получения флага, еще есть один способ получения brainfuck кода флага, при помощи следующего скрипта (путем применения операции OR со свдигом):

```python
from PIL import Image
import random
import sys
import string

img = Image.open('image-174.png')
img2 = Image.open('image-174_hehehe.png')
img_pix = img.convert('RGB')
img_pix2 = img2.convert('RGB')

width, height = img.size
width2, height2 = img2.size

print("Image 1: ", width, height)
print("Image 2: ", width, height)

line = ""
bitIndex = 7
ch = 0

for h in range(height):
  for w in range(width):
    r1, g1, b1 = img_pix.getpixel((w,h))
    r2, g2, b2 = img_pix2.getpixel((w,h))
    for p1, p2 in ((r1,r2),(g1,g2),(b1,b2),):   
       ch |= (abs(p1-p2) << bitIndex)
       bitIndex -= 1
       if bitIndex < 0:
          line += chr(ch)
          ch = 0
          bitIndex = 7

img.close()
img2.close()

print line
print len(line)
```

Вывод после выполнения кода следующий:

![VirtualBox_Kali2016.2_04_10_2016_22_51_04](https://github.com/maddevsio/publications/blob/master/ctf.com.ua/Suspicious_AVI/imgs/VirtualBox_Kali2016.2_04_10_2016_22_51_04.png)

**Автор: n0z3r0**