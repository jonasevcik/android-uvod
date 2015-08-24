
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
>File -> Settings -> Editor -> Code Style -> Java -> Code Generation -> Name prefix

>Field: *m*

>Static field: *s*