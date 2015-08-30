
# Základní stavební prvky


# Drawables
Drawable je klasický PNG obrázek, ale klidně i XML definice vektorového obrázku.


## Shape Drawables
Drawable si můžeme nadefinovat v XML. Výhodou je malá velikost a modifikovatelnost přímo v kódu. Výseldný obrázek skládáme z grafických primitiv, barev a přechodů.

Modrý obdélník se spodními kulatými rohy:
```xml
<shape android:shape="rectangle">
    <solid android:color="@android:color/blue_dark" />
    <corners android:bottomLeftRadius="8dp"
        android:bottomRightRadius="8dp" />
</shape>
```

Drawables můžeme tvořit více vrstvé a kombinovat tak základní prvky ve složité útvary:
```xml
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

<div style="text-align: center;">
    <img src="./img/2-9patch.png" alt="Barvy" style="width: 200px; box-shadow: none;" />
</div>

Jméno má podle rozdělení obrázku na 9 sektorů - "záplat":
* **1, 3, 7, 9** - rohy jsou statické a nenatahují se
* **2, 8** - horní a spodní hrany mohou být nataženy horizontálně
* **4, 6** - boční hrany mohou být nataženy vertikálně
* **5** - střed může být natažen horizontálně i vertikálně

<div style="text-align: center;">
    <img src="./img/2-9patch-1area.png" alt="1 oblast" style="width: 200px; box-shadow: none; margin-right: 20px;" />
    <img src="./img/2-9patch-2areas.png" alt="2 oblasti" style="width: 200px; box-shadow: none; margin-left: 20px;" />
</div>

Zelené oblasti se natahují v jednom směru. Růžové oblasti se natahují do obou stran. Oblastí pro natahování může být několik. To v případě, že nechcete deformovat části obrázku ve vnitřní části (ne na krajích).

### Optical bounds
Optical bounds byly zavedeny v Androidu 4.3. Určují oblast obrázku, která se "nepočítá" do jeho rozměrů. Toho se dá využít např. když je kolem útvaru 9patche nakreslen stín. Ten nechceme počítat do rozměrů útvaru.

<div style="text-align: center;">
    <img src="./img/2-9patch-optical-bounds.png" alt="Optical bounds" style="width: 200px; box-shadow: none;" />
</div>

### Draw 9-patch editor
[Editor](https://developer.android.com/tools/help/draw9patch.html) je součástí Android SDK. Umožňuje konvertovat PNG soubory na .9.png.


## Selectory
Selector je speciální drawable, která mění svůj vzhled na základě specifikovaných podmínek. Jedná se o seznam složený z drawables/barev a podmínek, kdy je zobrazit. Podmínek může být pro 1 stav specifikováno několik. Vyhodnocení podmínek probíhá odshora. Položka bez podmínek je tzv. fallback varianta.

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
   <item android:state_pressed="true" android:color="@color/light_blue"/>
   <item android:state_focused="true" android:color="@color/dark_blue"/>
   <item android:state_selected="true" android:state_activated="true" android:color="@color/light_blue_A400"/>
   <item android:state_selected="true" android:state_activated="false" android:color="@color/gray"/>
   <item android:color="@color/black"/>
</selector>
```


# Styly vs Témata
Účel stylů je především oddělit definice designu od samotného kódu obsahu - podobně jako CSS na webu. Samotné prvky GUI se dají vizuálně modifikovat přímo v layoutu, ale takový kód je nepřehledný a vznikají duplicitní definice. Např. v layoutu je 10 tlačítek a pro každé by byl definován stejný styl 10x.

Best practice je udržovat si definice v 2 souborech. theme.xml pro souhrn definic jednotlivých views tvořící dohromady téma celé aplikace. Dále styles.xml, ve kterém jsou již konkrétní definice pro jednotlivé prvky. Toto se dělá proto, že je třeba udržet hierarchii a přehlednost jednotlivých definic.

theme.xml
```xml
<style name="Theme" parent="android:Theme.Holo.Light">
   <item name="android:buttonStyle">@style/ButtonTheme</item>
   <item name="android:seekBarStyle">@style/SeekBarTheme</item>
</style>
```

styles.xml
```xml
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

Dědit lze 2 způsoby. Klíčovým slovem parent nebo tečkovou notací.

* Parent
```xml
<style name="Parent"/>
```
* Explicitní dědičnost
```xml
<style name="Child" parent="Parent"/>
```
* Implicitní dědičnost
```xml
<style name="Parent.Child"/>
```

Systémové styly jde dědit jen přes atribut parent. Pokud dědíme z vlastních stylů, můžeme použít jen tečkovou notaci. V příkladu *ButtonTheme.Big* má stejné atributy jako ButtonTheme, jen navíc mění velikost písma. Seznam všech [atributů](http://developer.android.com/reference/android/R.attr.html).

### Použití explicitní a implicitní dědičnosti zaráz
```xml
<style name="Implicit"/>
<style name="Explicit"/>
<style name="Implicit.Child" parent="Explicit"/>
```
Podědí Child jak ze stylu Implicit, tak ze stylu Explicit? **Ne**, při použití obou způsobů dědičnosti, dědí potomek pouze z explicitního rodiče. Proto raději používejte jen 1 způsob dědičnosti.

## Použití
Styly lze aplikovat na jednotlivé GUI elementy, na samostatné aktivity nebo na celou aplikaci.

MyLayout.xml
```xml
<Button style="@style/ButtonTheme.Big" />
```

Manifest.xml
```xml
<activity android:theme="@style/Theme">
```

Manifest.xml
```xml
<application android:theme="@style/Theme">
```

### Více stylů pro jeden View zaráz?
Na jeden View jde aplikovat pouze jeden styl. Výjimkou je pouze TextView, který jde ještě stylovat pomocí textAppearance.
```xml
<TextView
        android:textAppearance="@style/TextViewAppearance"
        style="@style/TextView"/>
```

Vždy u stylu textAppearance děďte z TextAppearance:

```xml
<style name="MyText" parent="TextAppearance.AppCompat">
    <item name="android:textColor">#fff</item>
</style>
```

## Holo
Dříve bylo styly je nejjednodušší dělat přes generátor. Pro všechny styly bez ActionBaru se dá použít Android Holo Colors Generator, jen pro ActionBar - Android Action Bar Style Generator.
* [Holo Colors](http://android-holo-colors.com/)
* [ActionBar style generator](http://jgilfelt.github.io/android-actionbarstylegenerator/)

Dnes už se nepoužívá. Vše směřuje k material designu. Původní přístup znamenal velké množství definic stylů a grafických resourců => nepřehledné, zabíralo místo.


## Material
Pokud použijeme *Theme.AppCompat*, můžeme jednoduše definicí několika základních barev upravit základní UI prvky.

```xml
<style name="Theme.MyTheme" parent="Theme.AppCompat.Light">
    <item name="colorPrimary">@color/material_blue_500</item>
    <item name="colorPrimaryDark">@color/material_blue_700</item>
    <item name="colorAccent">@color/material_green_A200</item>
</style>
```

<div style="text-align: center;">
    <img src="./img/2-colors.png" alt="Barvy" style="width: 300px;" />
</div>

Abyste nastylovali všechny prvky, použijte jejich AppCompat verzi:
<div>
<ul style="float: left;">
<li>AppCompatAutoCompleteTextView</li>
<li>AppCompatButton</li>
<li>AppCompatCheckBox</li>
<li>AppCompatCheckedTextView</li>
<li>AppCompatEditText</li>
</ul>

<ul style="float: right;">
<li>AppCompatMultiAutoCompleteTextView</li>
<li>AppCompatRadioButton</li>
<li>AppCompatRatingBar</li>
<li>AppCompatSpinner</li>
<li>AppCompatTextView</li>
</ul>
<div style="clear: both;"></div>
</div>

### Jak na to
* [Using the Material Theme](http://developer.android.com/training/material/theme.html)
* [Material Palette](http://www.materialpalette.com/) - Generování barev aplikace na základě výběru primární a doplňkové barvy

<div style="text-align: center;">
    <img src="./img/2-complementary-colors.png" alt="Doplňkové barvy" style="max-width: 380px; box-shadow: none;" />
</div>

## Layout
### Nepoužívejte RelativeLayout jako rodiče větvené hierarchie Views
RelativeLayout je dobrý pro jednoduchou hierarchii Views. U složitější může nastat problém u měření jeho rozměrů. RelativeLayout totiž používá 2 průchody měření. Při prvním proběhne přeměření jednotlivých jeho ChildViews a při druhém může na základě nich nastavit své rozměry.
* Google I/O 2013 - [Writing Custom Views for Android](https://www.youtube.com/watch?v=NYtB6mlu7vA&t=1m41s)
* Facebook's [Custom ViewGroups](https://sriramramani.wordpress.com/2015/05/06/custom-viewgroups/#more-406)


### Efektivní inflatování
Uvažujme layout:
```xml
<ViewGroup android:id="@+id/root">
    <View android:id="@+id/leaf" />
    <ViewGroup android:id="@+id/inner_group">
        <View android:id="@+id/inner_leaf" />
    </ViewGroup>
</ViewGroup>
```

**Rozdíl?**

1)
```java
ViewGroup vg = (ViewGroup)findViewById(R.id.inner_group);
View v = findViewById(R.id.inner_leaf);
```
2)
```java
ViewGroup vg = (ViewGroup)findViewById(R.id.inner_group);
View v = vg.findViewById(R.id.inner_leaf);
```

Pokud v Aktivitě voláme *findViewById()*, voláme tuto metodu od kořenového View postupně na všechny jeho ChildViews, dokud dané ID nenajdeme, nebo neprojdeme celou hierarchii. V případu 1 je postup prohledávání pro View v následovný:
> root->leaf  
> root->inner_group  
> inner_group->inner_leaf

V případu 2 je to jen:
> inner_group->inner_leaf


# Kam dál?
* [oficiální Google materiály](http://developer.android.com/training/index.html)
* mDevCamp 2013 [Optimalizace UI](https://www.youtube.com/watch?v=X_TJOSNzNug)
* [Nanodegree na Udacity](https://www.udacity.com/course/android-developer-nanodegree--nd801) - oficiální online kurz Androidu
* [Using styles and themes without going crazy](https://speakerdeck.com/dlew/using-styles-and-themes-without-going-crazy-1)