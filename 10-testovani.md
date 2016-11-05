# Testování
<div style="text-align: center;">
    <img src="./img/10-structure.png" alt="Anonymní třída" style="max-width: 600px;" />
</div>

## Unit/jednotkové testy
* jUnit – knihovna známá z klasické Javy, využívající asserty.
* Mockito – mokovací nástroj pro vytváření falešných objektů, které potřebujeme pro test, ale nejsou důležité pro vyhodnocení výsledku testu.
* PowerMock – podobné jako Mockito. Hodí se např. pro mokování statických tříd a androidího prostředí.

## UI testy
* Espresso - Je View viditelný? Má TextView nasetovanou správnou hlášku?
* UIAutomator - testování interakce v aplikaci. Zmáčkni button, odscrolluj...
