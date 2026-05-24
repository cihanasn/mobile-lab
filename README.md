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

Süper! Temel iskelet çalışıyor. Şimdi sıradaki adım — grid üzerine renkli noktaları yerleştirelim.

``` bash
package com.cihanasn.flowgame

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.view.View

// Bir noktayı temsil eder
data class Dot(val row: Int, val col: Int, val colorIndex: Int)

class GameView(context: Context) : View(context) {

    private val gridSize = 6 // 6x6 grid
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private var cellSize = 0f
    private var offsetX = 0f
    private var offsetY = 0f

    // Oyun renkleri
    private val colors = listOf(
        Color.parseColor("#e74c3c"), // kırmızı
        Color.parseColor("#3498db"), // mavi
        Color.parseColor("#2ecc71"), // yeşil
        Color.parseColor("#f39c12"), // turuncu
        Color.parseColor("#9b59b6")  // mor
    )

    // Level 1: 5 çift nokta, 6x6 grid
    private val dots = listOf(
        Dot(0, 0, 0), Dot(4, 4, 0), // kırmızı çift
        Dot(0, 5, 1), Dot(5, 0, 1), // mavi çift
        Dot(1, 1, 2), Dot(3, 3, 2), // yeşil çift
        Dot(0, 3, 3), Dot(5, 5, 3), // turuncu çift
        Dot(2, 0, 4), Dot(2, 5, 4)  // mor çift
    )

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
        drawDots(canvas)
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

    private fun drawDots(canvas: Canvas) {
        val radius = cellSize * 0.3f

        for (dot in dots) {
            val cx = offsetX + dot.col * cellSize + cellSize / 2f
            val cy = offsetY + dot.row * cellSize + cellSize / 2f

            // Dış halka
            paint.style = Paint.Style.STROKE
            paint.strokeWidth = 4f
            paint.color = colors[dot.colorIndex]
            canvas.drawCircle(cx, cy, radius, paint)

            // İç dolgu
            paint.style = Paint.Style.FILL
            paint.color = colors[dot.colorIndex]
            canvas.drawCircle(cx, cy, radius * 0.6f, paint)
        }
    }
}
```

<img width="428" height="875" alt="image" src="https://github.com/user-attachments/assets/e7c2fffd-987e-4dde-8fe0-2cf728f1c21e" />

ACTION_DOWN — Parmak ekrana değdi
- Parmak ekrana değdiğinde hangi hücreye dokunulduğunu buluyoruz
- O hücrede bir nokta var mı kontrol ediyoruz (dotAt)
- Varsa o rengi activeColorIndex olarak işaretliyoruz — "şu an bu rengi çiziyorum"
- O rengin yolunu sıfırlayıp başlangıç hücresini ekliyoruz
- invalidate() → "ekranı yeniden çiz" demek, bunu her değişiklikte çağırmak gerekiyor

ACTION_MOVE — Parmak sürükleniyor

``` bash
val prevIndex = currentPath.indexOf(cell)
if (prevIndex >= 0) {
    while (currentPath.size > prevIndex + 1) {
        currentPath.removeAt(currentPath.size - 1)
    }
}
```
Parmak daha önce geçtiği bir hücreye geri dönerse yolu o noktadan kesiyor — geri alma mekaniği bu

``` bash
val dr = Math.abs(cell.row - lastCell.row)
val dc = Math.abs(cell.col - lastCell.col)
if (dr + dc == 1) { ... }
```
dr + dc == 1 → sadece yatay veya dikey komşu hücreye geçişe izin veriyor, diagonal hareketi engelliyor
Örneğin: yukarı/aşağı/sağ/sol → dr+dc = 1 ✅
Çapraz → dr+dc = 2 ❌
“Diagonal hareket” çapraz hareket demektir.

``` bash
if (!cellOccupied(cell, activeColorIndex)) {
    val dotHere = dotAt(cell)
    if (dotHere == null || dotHere.colorIndex == activeColorIndex) {
        currentPath.add(cell)
    }
}
```
Başka bir rengin üzerinden geçemezsin (cellOccupied)
O hücrede farklı renkte bir nokta varsa oraya giremezsin
Aynı renkte nokta veya boş hücreyse yola ekle

ACTION_UP — Parmak kalktı

``` bash
MotionEvent.ACTION_UP -> {
    activeColorIndex = -1
    invalidate()
}
```

Sürükleme bitti, aktif rengi sıfırla
Ekranı son haliyle yeniden çiz

Kısaca onTouchEvent şunu yapıyor: parmak bir noktaya değince o rengi aktif eder, sürükleyince hücre hücre yolu büyütür, geri gidince yolu kısaltır, çakışmaları engeller, parmak kalkınca çizimi bitirir.

``` bash
package com.cihanasn.flowgame

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.graphics.Path
import android.view.MotionEvent
import android.view.View

// Bir noktayı temsil eder
data class Dot(val row: Int, val col: Int, val colorIndex: Int)
data class Cell(val row: Int, val col: Int)

class GameView(context: Context) : View(context) {

    private val gridSize = 6 // 6x6 grid
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private var cellSize = 0f
    private var offsetX = 0f
    private var offsetY = 0f

    // Oyun renkleri
    private val colors = listOf(
        Color.parseColor("#e74c3c"), // kırmızı
        Color.parseColor("#3498db"), // mavi
        Color.parseColor("#2ecc71"), // yeşil
        Color.parseColor("#f39c12"), // turuncu
        Color.parseColor("#9b59b6")  // mor
    )

    // Level 1: 5 çift nokta, 6x6 grid
    private val dots = listOf(
        Dot(0, 0, 0), Dot(4, 4, 0), // kırmızı çift
        Dot(0, 5, 1), Dot(5, 0, 1), // mavi çift
        Dot(1, 1, 2), Dot(3, 3, 2), // yeşil çift
        Dot(0, 3, 3), Dot(5, 5, 3), // turuncu çift
        Dot(2, 0, 4), Dot(2, 5, 4)  // mor çift
    )

    // Her renk için çizilen yol (hücre listesi)
    private val paths = mutableMapOf<Int, MutableList<Cell>>()

    // Şu an sürüklenen renk
    private var activeColorIndex = -1

    // Parmağın anlık pozisyonu (çizgi sürüklenirken)
    private var fingerX = 0f
    private var fingerY = 0f

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        cellSize = minOf(w, h).toFloat() / gridSize
        offsetX = (w - cellSize * gridSize) / 2f
        offsetY = (h - cellSize * gridSize) / 2f
    }

    // Piksel koordinatından grid hücresine dönüştür
    private fun pixelToCell(x: Float, y: Float): Cell? {
        val col = ((x - offsetX) / cellSize).toInt()
        val row = ((y - offsetY) / cellSize).toInt()
        if (row < 0 || row >= gridSize || col < 0 || col >= gridSize) return null
        return Cell(row, col)
    }

    // Hücre merkezinin piksel koordinatı
    private fun cellCenter(cell: Cell): Pair<Float, Float> {
        val cx = offsetX + cell.col * cellSize + cellSize / 2f
        val cy = offsetY + cell.row * cellSize + cellSize / 2f
        return Pair(cx, cy)
    }

    // O hücrede nokta var mı?
    private fun dotAt(cell: Cell): Dot? {
        return dots.find { it.row == cell.row && it.col == cell.col }
    }

    // O hücre başka bir rengin yolunda mı?
    private fun cellOccupied(cell: Cell, excludeColor: Int): Boolean {
        for ((colorIndex, path) in paths) {
            if (colorIndex == excludeColor) continue
            if (path.any { it.row == cell.row && it.col == cell.col }) return true
        }
        return false
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        val x = event.x
        val y = event.y
        val cell = pixelToCell(x, y)

        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                if (cell != null) {
                    val dot = dotAt(cell)
                    if (dot != null) {
                        // Noktaya dokunuldu, o rengin yolunu sıfırla ve başlat
                        activeColorIndex = dot.colorIndex
                        paths[activeColorIndex] = mutableListOf(cell)
                        fingerX = x
                        fingerY = y
                        invalidate()
                    }
                }
            }

            MotionEvent.ACTION_MOVE -> {
                if (activeColorIndex >= 0 && cell != null) {
                    val currentPath = paths[activeColorIndex] ?: return true
                    val lastCell = currentPath.lastOrNull()

                    if (lastCell != null && cell != lastCell) {
                        // Geri gidiyorsa yolu kısalt
                        val prevIndex = currentPath.indexOf(cell)
                        if (prevIndex >= 0) {
                            while (currentPath.size > prevIndex + 1) {
                                currentPath.removeAt(currentPath.size - 1)
                            }
                        } else {
                            // Komşu hücre mi kontrol et (diagonal değil)
                            val dr = Math.abs(cell.row - lastCell.row)
                            val dc = Math.abs(cell.col - lastCell.col)
                            if (dr + dc == 1) {
                                // Başka rengin üzerine geçme
                                if (!cellOccupied(cell, activeColorIndex)) {
                                    // Eğer farklı renkte bir noktaysa geçme
                                    val dotHere = dotAt(cell)
                                    if (dotHere == null || dotHere.colorIndex == activeColorIndex) {
                                        currentPath.add(cell)
                                    }
                                }
                            }
                        }
                    }
                    fingerX = x
                    fingerY = y
                    invalidate()
                }
            }

            MotionEvent.ACTION_UP -> {
                activeColorIndex = -1
                invalidate()
            }
        }
        return true
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.drawColor(Color.parseColor("#1a1a2e")) // koyu arka plan
        drawGrid(canvas)
        drawPaths(canvas)  // bunu ekle
        drawDots(canvas)
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

    private fun drawPaths(canvas: Canvas) {
        paint.style = Paint.Style.STROKE
        paint.strokeCap = Paint.Cap.ROUND
        paint.strokeJoin = Paint.Join.ROUND
        paint.strokeWidth = cellSize * 0.35f

        for ((colorIndex, path) in paths) {
            if (path.isEmpty()) continue
            paint.color = colors[colorIndex]

            val androidPath = Path()
            val (startX, startY) = cellCenter(path[0])
            androidPath.moveTo(startX, startY)

            for (i in 1 until path.size) {
                val (cx, cy) = cellCenter(path[i])
                androidPath.lineTo(cx, cy)
            }

            canvas.drawPath(androidPath, paint)
        }
    }

    private fun drawDots(canvas: Canvas) {
        val radius = cellSize * 0.3f

        for (dot in dots) {
            val cx = offsetX + dot.col * cellSize + cellSize / 2f
            val cy = offsetY + dot.row * cellSize + cellSize / 2f

            // Dış halka
            paint.style = Paint.Style.STROKE
            paint.strokeWidth = 4f
            paint.color = colors[dot.colorIndex]
            canvas.drawCircle(cx, cy, radius, paint)

            // İç dolgu
            paint.style = Paint.Style.FILL
            paint.color = colors[dot.colorIndex]
            canvas.drawCircle(cx, cy, radius * 0.6f, paint)
        }
    }
}
```

<img width="358" height="763" alt="image" src="https://github.com/user-attachments/assets/4ffc2ddc-0c10-448c-b15b-f032e21a4b32" />

