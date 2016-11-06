# Testování
<div style="text-align: center;">
    <img src="./img/10-structure.png" alt="Anonymní třída" style="max-width: 300px;" />
</div>

* test - jednotkové testy. Využívají jen Java SDK a jsou tedy spustitelné i na PC s JVM. 
* androidTest - zpřístupňují Context a umožňují testování tříd s návazností na Android. Pro spuštění vyžadují emulátor nebo zařízení s Androidem.


## Unit/jednotkové testy
* jUnit – knihovna známá z klasické Javy, využívající asserty.
* Mockito – mokovací nástroj pro vytváření falešných objektů, které potřebujeme pro test, ale nejsou důležité pro vyhodnocení výsledku testu.
* PowerMock – podobné jako Mockito. Hodí se např. pro mokování statických tříd a androidího prostředí.

## UI testy
* Espresso - Je View viditelný? Má TextView nasetovanou správnou hlášku?
* UIAutomator - testování interakce v aplikaci. Zmáčkni button, odscrolluj...

[Ukázkový projekt](https://github.com/jonasevcik/AndroidTestingDemo)
