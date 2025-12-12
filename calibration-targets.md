---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Калибрационни цели

MAPIR предлага различни калибрационни цели, които покриват широк спектър от приложения. Компактният T4-R50, показан по-долу, съдържа 4 панела, които са измерени за светлинно отражение от 250 до 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Дифузните референтни цели T4 имат следните криви на отражателната способност, [изтегляне на данни тук](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Отражателна способност :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Отражателна способност :: 400-1000 nm</p></figcaption></figure>Когато разгледате графиката на отражателната способност, можете да видите, че стойностите са дължина на вълната (ос x) спрямо процент на отражателната способност (ос y). Когато заснемем изображение на калибрационната цел, създаваме връзка между стойността на пиксела и процента на отражателната способност в рамките на спектъра, към който са чувствителни сензорите на камерата.

Това означава, че с всяко изображение, което заснемате с нашите камери, можете да използвате снимка на нашите рефлекторни цели, като [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) или [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), за да калибрирате изображенията за рефлекторност. След калибриране всеки пиксел в изображението е равен на процент отражателна способност.

Ако изведете калибрираните изображения в Chloros като типичен JPG или TIFF, процентът на отражателната способност се изчислява, като се раздели стойността на пиксела на битовата дълбочина на формата на изображението. Така че за JPG се разделя на 255, а за TIFF се разделя на 65 535. Можете също да изберете изходния формат PERCENT в Chloros, като тогава всеки пиксел ще варира от процентна стойност 0,0 до 1,0 (0% до 100% отражателна способност). Имайте предвид, че някои приложения за изображения не приемат изображения в проценти (с плаваща запетая) и те са с голям размер от гледна точка на съхранението.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
