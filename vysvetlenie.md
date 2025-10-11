Čo sú uniformné premenné?

Uniform = globálna premenná v shaderi, ktorá:

Je rovnaká pre všetky vertexy/fragmenty v jednom draw calle

Používa sa na: transformačné matice, farby, svetlá, textúry, čas, atď.

prečo preťažené metódy pretože uniformy majú rôzne dátové typy : 

Lebo uniformy majú rôzne dátové typy:

mat4 - transformačná matica
vec3 - pozícia svetla, farba
float - čas, alpha
int - textúra ID, boolean flag

validne pretazenie 

    void setUniform(const char* name, int value);
    void setUniform(const char* name, float value);
    void setUniform(const char* name, const glm::vec3& value);
    void setUniform(const char* name, const glm::mat4& value);

prečo je toto valídne preťaženie ? 

setUniform() s rovnakým názvom, ale rôznymi signatúrami:

Kompilátor automaticky vyberie správnu verziu na základe typu argumentu.

prakticky sa jedná o valídne preťaženie v c++ pretože názov metódy ostáva rovnaký mení sa ich ( parameter list ) = len počat arguemntov a ich datatype kompilátor to rozpozná na základe typov odovzdaných parametrov 


# Čo je to composite pattern ? 

návrhový vzor, ktorý umožnuje

zoskupovať objekty do stromovej štruktúry

jednotne pracovať s jednoduchými aj zloženými objektmi 

Kľúčové komponenty:

Component (abstraktná trieda/interface)

Definuje spoločné rozhranie pre všetky objekty
Deklaruje spoločnú operáciu operation()


Leaf (listový objekt)

Základný objekt bez potomkov
Implementuje operation() priamo


Composite (kompozitný objekt)

Objekt s potomkami
Obsahuje vector<Component*>
Implementuje operation() rekurzívne - zavolá operation() na všetkých potomkoch


Component : Transformation - base class 
Leaf - translate, scale, rotation 
compsoite - composite 
Operation - getmatrix 


// Transformation.h
class Transformation
{
protected:
    glm::mat4 matrix;  // Spoločný atribút

public:
    Transformation();
    virtual glm::mat4 getMatrix();  // ✅ Spoločná operácia
};

=== toto je component, definuje 

Úloha:

Definuje spoločné rozhanie pre všetky transformácie
getMatrix() - virtuálna metóda (môže byť preťažená)




// Translate.h
class Translate : public Transformation
{
public:
    Translate(float x, float y, float z)
    {
        matrix = glm::translate(glm::mat4(1.0f), glm::vec3(x, y, z));
    }
};

// Rotate.h
class Rotate : public Transformation
{
public:
    Rotate(float angle, float x, float y, float z)
    {
        matrix = glm::rotate(glm::mat4(1.0f), glm::radians(angle), glm::vec3(x, y, z));
    }
};

// Scale.h
class Scale : public Transformation
{
public:
    Scale(float x, float y, float z)
    {
        matrix = glm::scale(glm::mat4(1.0f), glm::vec3(x, y, z));
    }
};

toto sú leafy 

Charakteristika Leaf:

✅ Jednoduché objekty bez potomkov
✅ Implementujú getMatrix() priamo (z Transformation base class)
✅ Vypočítajú maticu v konštruktore
✅ Nemôžu obsahovať ďalšie transformácie




// Composite.h
class Composite : public Transformation
{
private:
    vector<Transformation*> transformations;  // ✅ Obsahuje potomkov!

public:
    Composite();
    
    void addTransformation(Transformation* transformation);
    
    // ✅ REKURZÍVNA implementácia!
    glm::mat4 getMatrix() override
    {
        glm::mat4 result = glm::mat4(1.0f);  // Identita
        
        for (Transformation* t : transformations)
        {
            result = result * t->getMatrix();  // ✅ Rekurzívne volanie!
        }
        
        return result;
    }
    
    void clear() { transformations.clear(); }
};


✅ Zložený objekt obsahujúci vector<Transformation*>
✅ addTransformation() - pridáva potomkov
✅ getMatrix() je rekurzívne - volá getMatrix() na všetkých potomkoch
✅ Môže obsahovať Leaf aj ďalšie Composite objekty!


aké to prináša výhody ? 

jednotné rozhranie 
flexibilita 
znovupouzirelnost
rekurzivne spracovanie 

## Single responsibility princíp - SRP 

vztahy : 

2.1 Application → SceneManager (KOMPOZÍCIA)


Typ vzťahu: KOMPOZÍCIA (Composition)

Application vlastní SceneManager
Keď sa zmaže Application, zmaže sa aj SceneManager
Životnosť: SceneManager závisí na Application


2.2 SceneManager → Scene (AGREGÁCIA)

Typ vzťahu: AGREGÁCIA (Aggregation)

SceneManager spravuje scény, ale nevlastní ich pamäť
Scény môžu existovať aj bez SceneManagera
Životnosť: Scény sú nezávislé (vytvorené zvonku)

2.3 Scene → DrawableObject (AGREGÁCIA)

Typ vzťahu: AGREGÁCIA (Aggregation)

Scene spravuje objekty, ale nevlastní ich pamäť
Objekty môžu existovať aj bez Scene
Životnosť: Objekty sú nezávislé

Mentálny model fajn analógia : 

Application          = Riaditeľ knižnice (vlastní budovu)
SceneManager         = Katalógový systém (eviduje knihy)
Scene                = Polica s knihami
DrawableObject       = Jednotlivá kniha


## Vysvetlenie konceptu observer 


```plain
Subject (abstraktná trieda)
├── attach(Observer)     - pridá observera do zoznamu
├── detach(Observer)     - odstráni observera
└── notify()             - upozorní všetkých observerov

Observer (interface)
└── update()             - metóda volaná pri notifikácii

ConcreteSubject          ConcreteObserver
├── getState()           └── update()
└── setState()               └── observerState = subject->getState()


```

Ako to funguje:

Subject (Pozorovateľný) drží zoznam Observerov
Keď sa zmení stav Subjectu (napr. pozícia kamery), zavolá notify()
notify() prejde všetkých observerov a zavolá ich update()
Každý Observer si v update() vytiahne nový stav zo Subjectu


2. Vzťah medzi Camera a ShaderProgram v našom projekte

Prečo potrebujeme Observer?

Problém

Keď sa hýbe kamera (WSAD, myš), mení sa jej view matrix
Shader potrebuje túto view matrix na správne vykreslenie scény
Bez observera by sme museli manuálne volať shaderProgram->setViewMatrix() po každom pohybe kamery

Riešenie:

Camera je Subject (ConcreteSubject)
ShaderProgram je Observer (ConcreteObserver)
Keď sa kamera pohne → automaticky notifikuje shader → shader si aktualizuje view matrix

Konkrétna implementácia:


```plain 

Camera (Subject)                    ICameraObserver (Interface)
├── observers[]                     └── update(Camera*)
├── attach(ICameraObserver*)        
├── detach(ICameraObserver*)        ShaderProgram (Observer)
├── notify()                        └── update(Camera* cam)
└── moveForward()                       └── setMatrix("view", cam->getViewMatrix())
    └── notify()  // <- zavolá sa!




```

3️⃣ Môj navrhovaný prístup: Jedna kamera + statická view matrix

✅ Výhody:

Optimálne využitie pamäte: Statická view matrix je len glm::mat4 (64 bytes)
Zachováva pozíciu: Scéna 7 → 6 → 7 uchováva pozíciu kamery
Efektívne: Observer pattern pracuje len v scéne 7
Flexibilný reset: Môžeš resetovať kameru len pri 1-6 → 7, nie pri 7 → 6 → 7
Čitateľný kód: Jasne vidno, kedy sa používa statická vs. dynamická view matrix
Škálovateľné: Ľahko pridáš ďalšie 3D scény, prakticky a len vytvoríme buď novú inštanciu alebo budeme len zdielat zo sceny 7 

❌ Nevýhody:

Mierne komplexnejšia logika prepínania (ale iba o pár riadkov)

Zložitosť: 🟡 Nízka-Stredná, UX: 🟢 Dobrý

------

alternatívny prístup 

Camera* staticCamera;   // Pre scény 1-6
Camera* dynamicCamera;  // Pre scénu 7

✅ Výhody:

Jasná separácia zodpovedností
Každá kamera má svoj stav
Pozícia v scéne 7 sa automaticky zachováva

❌ Nevýhody:

Zbytočná pamäť: Statická kamera sa nikdy nepohybuje → zbytočná inštancia
Duplicita observerov: Musíš registrovať obe kamery k shader programom
Komplikovanejšie prepínanie: Musíš manuálne upravovať, ktorá kamera je aktívna
Neefektívne: Statická kamera volá notify() aj keď sa nič nemení

---- 
bugs : 

🔍 Prečo to predtým nefungovalo?
Problém č. 1: Neinicializovaná viewMatrix v konštruktore
Pôvodný kód v Camera.cpp:

Camera::Camera(glm::vec3 pos, glm::vec3 up, float yaw, float pitch)
    : position(pos), worldUp(up), yaw(yaw), pitch(pitch),
      movementSpeed(2.5f), mouseSensitivity(0.1f)
{
    front = glm::vec3(0.0f, 0.0f, -1.0f);
    updateCameraVectors();
    // ❌ viewMatrix NIE JE inicializovaná!
}

Čo sa stalo:

Kamera sa vytvorí s pozíciou (0, 1, 5)
updateCameraVectors() vypočíta front, right, up vektory
ALE viewMatrix ostáva neinicializovaná → obsahuje garbage data (náhodné hodnoty z pamäte)

Problém č. 2: View matrix sa aktualizovala len pri pohybe
Pozri sa na Camera::notify():
cppvoid Camera::notify()
{
    // ✅ TU sa viewMatrix aktualizuje
    viewMatrix = glm::lookAt(position, position + front, up);
    
    // Notifikuj observerov
    for (ICameraObserver* observer : observers)
    {
        observer->update(this);
    }
}
Kedy sa volala notify()?

❌ NIE pri vytvorení kamery (konštruktor nevolá notify())
✅ Len pri pohybe kamery (moveForward, moveBackward, atď.)
✅ Len pri rotácii myšou (processMouseMovement)





