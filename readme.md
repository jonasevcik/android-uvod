
# Toto je Android!
Android běží na zařízeních s mnohdy slabým CPU a málo RAM, proto pro něj nemusí platit best practices, které znáte z Javy. I zvyklosti v pojmenovávání jsou jiné.

## Pojmenování
Privátní atributy mají prefix *m*. Statické atributy mají prefix *s*.

```
public class Something {
   private Object mObject;
   private static String sString;
 
   public void setObject(Object object) {
      mObject = object;
   }
 
   public Object getObject() {
      return mObject;
   }
}
```


## Android Studio
Pro snažší práci s pojmenováváním a generováním getterů/setterů si nastavte:
> File -> Settings -> Editor -> Code Style -> Java -> Code Generation -> Name prefix

> Field: *m*

> Static field: *s*


## Zvýraznění anonymních tříd
Anonymní třídy jsou častým zdrojem memory leaků, protože nemusí být viditelné, že přistupujete k vnějším objektům. Anonymní třída si na ně drží neviditelné reference. To působí problémy především u objektů s životním cyklem (Aktivity, Fragmenty...), které v tom případě nemohou být garbage collectovány.

Pro upomenutí si zvolte výraznou barvu pro anonymní třídy:
> File -> Settings -> Editor -> Colors & Fonts -> Java -> Anonymous class


## Cizí styly pro import
Vyzkoušejte, jak vám vyhovují nastavení jiných:

* Google
* Square
* Avast