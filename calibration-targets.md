---
description: Laboratóriumban mért panelek, amelyeket a rögzített adatok utólagos feldolgozásához kalibrálnak
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Kalibrációs célok

Az MAPIR különböző kalibrációs célokat kínál, hogy lefedje a különböző alkalmazási területeket. Az alábbi kompakt T4-R50 4 panelt tartalmaz, amelyek fényvisszaverődését 250–2500 nm tartományban mérték.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

A T4 diffúz referencia célok a következő visszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 visszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 visszaverődés :: 400–1000 nm</p></figcaption></figure>

A visszaverődési grafikonon látható, hogy az értékek a hullámhossz (x-tengely) és a visszaverődési százalék (y-tengely) függvényében vannak ábrázolva. Amikor képet készítünk a kalibrációs célpontról, akkor a kamera érzékeny szenzorcsatornáinak spektrumán belül kapcsolatot hozunk létre a pixelérték és a visszaverődési százalék között.

Ez azt jelenti, hogy minden, kameráinkkal készített képpel használhatja a fényvisszaverődési célpontjaink fényképét, például a [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) vagy a [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) fényképet a képek fényvisszaverődésének kalibrálásához. A kalibrálás után a kép minden egyes képpontja egyenlő a visszaverődés százalékával.

Ha a kalibrált képeket Chloros formátumban adja ki, mint a tipikus JPG vagy TIFF, akkor a visszaverődési százalékot a képpont értékének és a képformátum bitmélységének hányadosaként számítja ki. Tehát JPG esetén 255-tel, TIFF esetén pedig 65 535-tel kell osztani. Választhatja a PERCENT formátumú kimenetet is az Chloros-ben, és akkor minden pixel 0,0 és 1,0 közötti százalékos értéket fog kapni (0% és 100% közötti visszaverődés). Ne feledje, hogy egyes képszerkesztő programok nem fogadják el a százalékos (lebegőpontos) képeket, és ezek tárolási szempontból nagy méretűek.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
