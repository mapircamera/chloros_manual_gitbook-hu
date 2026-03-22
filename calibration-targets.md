---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrációs célpontok

Az MAPIR számos alkalmazási területet lefedő, különféle kalibrációs célpontokat kínál. Az alábbi képen látható kompakt T4-R50 modell 4 panelt tartalmaz, amelyek fényvisszaverődését 250–2500 nm hullámhossz-tartományban mérték meg.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>A T4 diffúz referencia célpontok a következő fényvisszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 fényvisszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflektancia :: 400–1000 nm</p></figcaption></figure>A T4P diffúz referencia célpontok a következő visszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P fényvisszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflektancia :: 400-1000 nm</p></figcaption></figure>A reflektancia-grafikonon látható, hogy az értékek a hullámhossz (x-tengely) és a reflektancia százalékos aránya (y-tengely) függvényében vannak ábrázolva. Amikor képet készítünk a kalibrációs célpontról, akkor létrehozunk egy kapcsolatot a pixelérték és a reflektancia százalékos aránya között, azon a spektrumon belül, amelyre a kamera egyes szenzorsávjai érzékenyek.

Ez azt jelenti, hogy minden, kameráinkkal rögzített kép esetén felhasználhatja a fényvisszaverő célpontjaink, például a [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) vagy a [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) fényképét a képek fényvisszaverődésének kalibrálásához. A kalibrálás után a kép minden egyes képpontja megegyezik a visszaverődés százalékával.

Ha a kalibrált képeket Chloros formátumban szokásos JPG-ként vagy TIFF formátumban adja ki, akkor a reflektancia százalékát úgy számítja ki, hogy a pixelértéket elosztja a képformátum bitmélységével. Tehát JPG esetén 255-tel, TIFF esetén pedig 65 535-tel kell osztani. Választhatja az Chloros formátumú PERCENT kimenetet is, amely esetben minden pixel 0,0 és 1,0 közötti százalékos értéket fog felvenni (0% és 100% közötti fényvisszaverődés). Ne feledje azonban, hogy egyes képszerkesztő programok nem támogatják a százalékos (lebegőpontos) képeket, és ezek tárolási szempontból nagy méretűek.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
