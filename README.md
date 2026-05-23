# mobile-lab - Flow Free

👉 https://developer.android.com/studio adresine gir ve "Download Android Studio" butonuna tıkla.

Kurulum sırasında gelen ekranda "Android Virtual Device" seçeneği işaretliyse bırak, değilse işaretle.

C:\Program Files\Android\Android Studio

Android Studio Setup Wizard

"Next" ile devam et — "Install Type" ekranında "Standard" seç. Bu seçenek SDK, emülatör ve gerekli her şeyi otomatik kurar.

Şimdi "Finish" butonuna tıkla — Android Studio ana ekranı (Welcome ekranı) açılacak.

"New Project" a tıkla, açılan ekranda template seçimi gelecek.

"Empty Activity" seç ve Next'e tıkla.

Şimdi proje ayarları ekranı geldi. Şöyle doldur:

Name: FlowGame (ya da istediğin bir isim)
Package name: com.cihanasn.flowgame
Save location: varsayılan kalabilir
Language: Kotlin
Minimum SDK: API 26 ("Oreo"; Android 8.0)

Doldurunca "Finish" e tıkla.

Proje oluşturuluyor, biraz sürebilir — altta Gradle sync çalışıyor olacak. Tamamlanınca sol tarafta dosya ağacı ve ortada MainActivity.kt açılmış olmalı.

Ortam hazır. Şimdi emülatör oluşturalım ki oyunu test edebilelim.
Üst menüden Tools → Device Manager a tıkla

"Add a new device" e tıkla, sonra "Create Virtual Device" seç.
Cihaz listesi gelecek — orada Pixel 6 u seç ve Next'e tıkla.

<img width="1138" height="877" alt="image" src="https://github.com/user-attachments/assets/70ed26c8-96ad-4f8f-8d39-706f8d329d48" />

"Finish" e tıkla, indirme otomatik başlayacaktır.

İndirme bitince emülatör listeye eklenecek ve artık oyunu çalıştırmaya hazır olacağız.

 Artık ortam tamamen hazır. Özet olarak şu ana kadar:

✅ Android Studio kuruldu
✅ SDK ve emülatör araçları kuruldu
✅ FlowGame projesi oluşturuldu
✅ Pixel 6 API 36 emülatörü hazır

Şimdi emülatörün çalışıp çalışmadığını test edelim. Üst toolbar'da yeşil ▶ Run butonuna tıkla — emülatör açılıp boş "Hello World" ekranı gelmeli.

<img width="393" height="838" alt="image" src="https://github.com/user-attachments/assets/0aaf8e1d-f11a-45c1-bf25-f6e14dfb296f" />

Android Studio'da sol taraftaki dosya ağacında app → java → com.cihanasn.flowgame klasörüne sağ tıkla → New → Kotlin Class/File de.

``` bash
package com.cihanasn.flowgame

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.view.View

class GameView(context: Context) : View(context) {

    private val gridSize = 6 // 6x6 grid
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private var cellSize = 0f
    private var offsetX = 0f
    private var offsetY = 0f

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        cellSize = minOf(w, h).toFloat() / gridSize
        offsetX = (w - cellSize * gridSize) / 2f
        offsetY = (h - cellSize * gridSize) / 2f
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.drawColor(Color.parseColor("#1a1a2e")) // koyu arka plan
        drawGrid(canvas)
    }

    private fun drawGrid(canvas: Canvas) {
        paint.color = Color.parseColor("#2a2a4a")
        paint.strokeWidth = 2f
        paint.style = Paint.Style.STROKE

        for (row in 0..gridSize) {
            // yatay çizgiler
            canvas.drawLine(
                offsetX,
                offsetY + row * cellSize,
                offsetX + gridSize * cellSize,
                offsetY + row * cellSize,
                paint
            )
            // dikey çizgiler
            canvas.drawLine(
                offsetX + row * cellSize,
                offsetY,
                offsetX + row * cellSize,
                offsetY + gridSize * cellSize,
                paint
            )
        }
    }
}
```
canvas bizim çizim tahtamız.

Önce arka planı koyu lacivert yapıyoruz
Sonra drawGrid i çağırıyoruz

offsetX + row * cellSize → her çizginin başlangıç noktasını hesaplar

Kısacası şu an sadece koyu arka plan üzerinde 6x6 boş bir grid çiziyor. Henüz renkli nokta yok, dokunma yok — sadece temel iskelet.

Önce GameView'ı MainActivity.kt'e bağlamamız lazım, yoksa ekranda görünmez.
MainActivity.kt'i aç ve içeriği şununla değiştir:

``` bash
package com.cihanasn.flowgame

import android.app.Activity
import android.os.Bundle

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(GameView(this))
    }
}
```

build.gradle.kts (Module: app) dosyasını aç ve dependencies bloğu

``` bash
implementation("androidx.appcompat:appcompat:1.7.0")
```

Ekledikten sonra üstte sarı bar çıkacak "Sync Now" a tıkla, sync bitince ▶ Run a bas.

override fun onCreate(...) → Activity ilk oluşturulduğunda Android bu metodu çağırır. Tüm başlangıç işlemleri buraya yazılır.
setContentView(GameView(this)) → ekrana ne gösterileceğini söylüyoruz.

<img width="405" height="845" alt="image" src="https://github.com/user-attachments/assets/7253191b-d91b-4bc7-9152-b3b92973f3bd" />


