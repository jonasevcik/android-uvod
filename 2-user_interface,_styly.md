
# Základní stavební prvky


# Drawables
Drawable je klasický PNG obrázek, ale klidně i XML definice vektorového obrázku.


## Shape Drawables
Drawable si můžeme nadefinovat v XML. Výhodou je malá velikost a modifikovatelnost přímo v kódu. Výseldný obrázek skládáme z grafických primitiv, barev a přechodů.

Modrý obdélník se spodními kulatými rohy:
```
<shape android:shape="rectangle">
    <solid android:color="@android:color/blue_dark" />
    <corners android:bottomLeftRadius="8dp"
        android:bottomRightRadius="8dp" />
</shape>
```

Drawables můžeme tvořit více vrstvé a kombinovat tak základní prvky ve složité útvary:
```
<layer-list>
    <item>
        <shape>
            <solid android:color="@color/gray" />
            <corners android:bottomRightRadius="4dp"
                android:bottomLeftRadius="4dp"
                android:topRightRadius="4dp"
                android:topLeftRadius="4dp" />
        </shape>
    </item>
    <item android:bottom="4dp">
        <shape>
            <solid android:color="@color/black" />
            <corners android:bottomRightRadius="4dp"
                android:bottomLeftRadius="4dp"
                android:topRightRadius="4dp"
                android:topLeftRadius="4dp" />
        </shape>
    </item>
</layer-list>
```


## 9-patch
Speciální formát PNG obrázku ([.9.png](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch)), kde jsou čárami specifikovány oblasti obrázku, které je možno natahovat.


## Selectory
Selector je speciální drawable, která mění svůj vzhled na základě specifikovaných podmínek. Jedná se o seznam složený z drawables/barev a podmínek, kdy je zobrazit. Podmínek může být pro 1 stav specifikováno několik. Vyhodnocení podmínek probíhá odshora. Položka bez podmínek je tzv. fallback varianta.

```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
   <item android:state_pressed="true" android:color="@color/light_blue"/>
   <item android:state_focused="true" android:color="@color/dark_blue"/>
   <item android:state_selected="true" android:state_activated="true" android:color="@color/light_blue_A400"/>
   <item android:state_selected="true" android:state_activated="false" android:color="@color/gray"/>
   <item android:color="@color/black"/>
</selector>
```


# Styly
Účel stylů je především oddělit definice designu od samotného kódu obsahu - podobně jako CSS na webu. Samotné prvky GUI se dají vizuálně modifikovat přímo v layoutu, ale takový kód je nepřehledný a vznikají duplicitní definice. Např. v layoutu je 10 tlačítek a pro každé by byl definován stejný styl 10x.

Best practice je udržovat si definice v 2 souborech. theme.xml pro souhrn definic jednotlivých views tvořící dohromady téma celé aplikace. Dále styles.xml, ve kterém jsou již konkrétní definice pro jednotlivé prvky. Toto se dělá proto, že je třeba udržet hierarchii a přehlednost jednotlivých definic.

theme.xml
```
<style name="Theme" parent="android:Theme.Holo.Light">
   <item name="android:buttonStyle">@style/ButtonTheme</item>
   <item name="android:seekBarStyle">@style/SeekBarTheme</item>
</style>
```

styles.xml
```
<style name="ButtonTheme" parent="android:Widget.Holo.Light.Button">
   <item name="android:background">@drawable/theme_btn_default_holo_light</item>
   <item name="android:textColor">@color/white</item>
   <item name="android:textSize">@dimen/button_text</item>
 </style>
 
 <style name="ButtonTheme.Big">
   <item name="android:textSize">@dimen/button_text_big</item>
 </style>
 
 <style name="SeekBarTheme" parent="android:Widget.Holo.Light.SeekBar">
   <item name="android:progressDrawable">@drawable/theme_scrubber_progress_horizontal_holo_light</item>
   <item name="android:indeterminateDrawable">@drawable/theme_scrubber_progress_horizontal_holo_light</item>
   <item name="android:thumb">@drawable/theme_scrubber_control_selector_holo_light</item>
 </style>
```


## Dědičnost
Práci se styly si můžeme zjednodušit dědičností. Např. máme definovaný kompletní styl pro tlačítko (např. 10 atributů) a chceme vyrobit nový styl, který se liší pouze v 1 atributu. Nemusíme definovat celý nový styl ale určíme si ten původní jako rodič a definujeme jen lišící se atribut.

Dědit lze 2 způsoby. Klíčovým slovem parent nebo tečkovou notací. Např. náš *ButtonTheme* dědí od *android:Widget.Holo.Light.Button*

```
<style name="ButtonTheme" parent="android:Widget.Holo.Light.Button">
```

Systémové styly jde dědit jen přes parent. Pokud dědíme z vlastních stylů, můžeme použít jen tečkovou notaci. V příkladu *ButtonTheme.Big* má stejné atributy jako ButtonTheme, jen navíc mění velikost písma. Seznam všech [atributů](http://developer.android.com/reference/android/R.attr.html).


## Použití
Styly lze aplikovat na jednotlivé GUI elementy, na samostatné aktivity nebo na celou aplikaci.

MyLayout.xml
```
<Button style="@style/ButtonTheme.Big" />
```

Manifest.xml
```
<activity android:theme="@style/Theme">
```

Manifest.xml
```
<application android:theme="@style/Theme">
```


## Holo
Dříve bylo styly je nejjednodušší dělat přes generátor. Pro všechny styly bez ActionBaru se dá použít Android Holo Colors Generator, jen pro ActionBar - Android Action Bar Style Generator.
* [Holo Colors](http://android-holo-colors.com/)
* [ActionBar style generator](http://jgilfelt.github.io/android-actionbarstylegenerator/)

Dnes už se nepoužívá. Vše směřuje k material designu. Původní přístup znamenal velké množství definic stylů a grafických resourců => nepřehledné, zabíralo místo.


## Material
Pokud použijeme *Theme.AppCompat*, můžeme jednoduše definicí několika základních barev upravit základní UI prvky.

```
<style name="Theme.MyTheme" parent="Theme.AppCompat.Light">
    <item name="colorPrimary">@color/material_blue_500</item>
    <item name="colorPrimaryDark">@color/material_blue_700</item>
    <item name="colorAccent">@color/material_green_A200</item>
</style>
```
Abyste nastylovali všechny prvky, použijte jejich AppCompat verzi:
* AppCompatAutoCompleteTextView
* AppCompatButton
* AppCompatCheckBox
* AppCompatCheckedTextView
* AppCompatEditText
* AppCompatMultiAutoCompleteTextView
* AppCompatRadioButton
* AppCompatRatingBar
* AppCompatSpinner
* AppCompatTextView

### Jak na to
* [Using the Material Theme](http://developer.android.com/training/material/theme.html)
* [Material Palette](http://www.materialpalette.com/) - Generování barev aplikace na základě výběru primární a doplňkové barvy

