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
```cpp

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
```
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



🎯 Zhrnutie - Prečo shadery potrebujú view matrix?


1. View matrix definuje, čo kamera vidí

Objekt vo world space: (5, 0, -10)
Kamera na: (0, 1, 3)

↓ View matrix

Objekt v camera space: (5, -1, -13)  // Relatívne ku kamere

2. Keď sa kamera pohne, view matrix sa zmení


Kamera NA: (0, 1, 3)  → viewMatrix A
Kamera NA: (0, 1, 0)  → viewMatrix B  // INÁ MATRIX!


3. Každý shader potrebuje AKTUÁLNU view matrix



// Vertex shader:
gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(vp, 1.0);
//                               ^^^^^^^^^^^
//                               Ak je stará → objekt na zlom mieste!
//                               Ak je nová → objekt na správnom mieste!


4. Observer pattern automatizuje aktualizáciu


Pohyb kamery → notify() → update všetkých shaderov → všetci majú NOVÚ view matrix


1. Kamera sa POHNE (position sa zmení)
   ↓
2. notify() sa ZAVOLÁ
   ↓
3. View matrix sa PREPOČÍTA (viewMatrix = glm::lookAt(...))
   ↓
4. UPOVEDOMIA sa VŠETCI observers
   ↓
5. Každý observer (shader) si VYTIAHNE novú view matrix
   ↓
6. HOTOVO! ✅


void Camera::notify()
{
    // 1. Prepočítaj view matrix
    viewMatrix = glm::lookAt(position, position + front, up);
    
    // 2. Upovedom observerov
    for (ICameraObserver* observer : observers)
    {
        observer->update(this);  // <- Pošlem JEN pointer na seba!
        //                ^^^^
        //                NIE viewMatrix priamo, ale CELÝ Camera objekt!
    }
}

// 3. Observer si SÁM vytiahne view matrix
void ShaderProgram::update(Camera* camera)
{
    use();
    setUniform("viewMatrix", camera->getViewMatrix());
    //                       ^^^^^^^^^^^^^^^^^^^^^^^^
    //                       ZAVOLÁM metódu getViewMatrix() - observer si to vezme SÁM!
}


Spôsob B: "Pull" (vyťahovanie dát) ← TVOJ KÓD! ✅

void Camera::notify()
{
    viewMatrix = glm::lookAt(...);
    
    for (ICameraObserver* observer : observers)
    {
        observer->update(this);  // <- Pošlem pointer na CELÚ kameru
        //                ^^^^
    }
}

void ShaderProgram::update(Camera* camera)
{
    use();
    setUniform("viewMatrix", camera->getViewMatrix());  // Vytiahnem si to SÁM
    //                       ^^^^^^^^^^^^^^^^^^^^^^^^
    
    // BONUS: Môžem si vytiahnut ČO CHCEM!
    setUniform("cameraPos", camera->getPosition());
    setUniform("cameraDir", camera->getFront());
}

🧠 Tvoj kód používa "Pull" pattern:
observer->update(this);  // "Hej, niečo sa zmenilo, tu máš prístup ku mne!"

camera->getViewMatrix();  // Chcem view matrix
camera->getPosition();     // Chcem pozíciu
camera->getFront();        // Chcem smer

// Tvoj kód:
Camera* camera = new Camera();
ShaderProgram* shader1 = new ShaderProgram(...);

camera->attach(shader1);  // ← "Odber" na kameru!
// Teraz je shader1 v zozname observers

camera->moveForward();  // ← Kamera sa pohla!
// ↓
notify() {
    for (ICameraObserver* obs : observers)
        obs->update(this);  // ← Shader dostane update!
}


📌 Registrácia - detailný rozpis

camera->attach(shaderProgram1);

// 1. VOLANIE metódy attach() na Camera objekte
camera->attach(shaderProgram1);
//      ^^^^^^ Metóda Camera::attach()
//             ^^^^^^^^^^^^^^^ Pointer na ShaderProgram objekt (napr. 0x1000)

// 2. VNÚTRI metódy attach():
void Camera::attach(ICameraObserver* observer)  // observer = 0x1000
{
    observers.push_back(observer);  // Pridaj pointer do vectora
    //        ^^^^^^^^^
    //        Systémová funkcia std::vector::push_back()
}

// 3. VÝSLEDOK:
// observers = [0x1000]  // Vector obsahuje pointer na shaderProgram1


------------


špecifiká pre pointre v jazyku c++ 

ShaderProgram* shader1 = new ShaderProgram(...);
//             ^^^^^^^ Pointer (adresa objektu v pamäti, napr. 0x1000)

camera->attach(shader1);
//             ^^^^^^^ Posielam ADRESU, nie kópiu objektu!

// observers = [0x1000]  // Vector ukladá ADRESU!

prečo pointre ? 

std::vector<ShaderProgram> observers;  // Vector OBJEKTOV, nie pointerov!

observers.push_back(shaderProgram1);  // ❌ SKOPÍRUJE celý objekt!
// observers = [KÓPIA shaderProgram1]

// PROBLÉM:
for (ShaderProgram& obs : observers) {
    obs.update(camera);  // Aktualizuješ KÓPIU, nie originál! ❌
}

std::vector<ICameraObserver*> observers;  // Vector POINTEROV!

observers.push_back(shaderProgram1);  // ✅ Uloží len ADRESU (8 bytov)
// observers = [0x1000]  // Pointer na originál!

// SPRÁVNE:
for (ICameraObserver* obs : observers) {
    obs->update(this);  // Aktualizuješ ORIGINÁL! ✅
}


 Výhody const:

Bezpečnosť - Observer NEMÔŽE omylom zmeniť Subject
Jasnosť - Kód jasne hovorí: "Len čítam, nič nemeňím"
Kompilátor ťa chráni - Pokus o modifikáciu = CHYBA pri kompilácii

// BEZ const (nebezpečné):
void update(Camera* camera) {
    camera->position = glm::vec3(0,0,0);  // ✅ Ide to, ale je to ZLÉ!
}

// S const (bezpečné):
void update(const Camera* camera) {
    camera->position = glm::vec3(0,0,0);  // ❌ CHYBA KOMPILÁTORA!
    camera->getViewMatrix();              // ✅ OK, len čítaš
}




prepínanie medzi scénami samotnými 


void Application::key_callback(GLFWwindow* window, int key, int scancode, 
                               int action, int mods)
{
    if (action == GLFW_PRESS)
    {
        if (key >= GLFW_KEY_1 && key <= GLFW_KEY_7)
        {
            int sceneIndex = key - GLFW_KEY_1;  // 0-6
            sceneManager->switchToScene(sceneIndex);

            // ========== SUPER JEDNODUCHÁ LOGIKA ==========
            if (sceneIndex == 6)  // Prepnutie NA scénu 7
            {
                firstMouse = true;  // Reset myši
                
                // ✅ MANUÁLNE prepíš view matrix na dynamickú (z kamery)
                shaderProgram1->use();
                shaderProgram1->setUniform("viewMatrix", camera->getViewMatrix());
                shaderProgram2->use();
                shaderProgram2->setUniform("viewMatrix", camera->getViewMatrix());
                // ... to isté pre všetky shadery ...
            }
            else  // Prepnutie NA scény 1-6
            {
                restoreStaticViewMatrix();  // ✅ Obnov statickú view matrix
            }
        }
    }
}

Čo je view matrix a prečo ju potrebujeme ? 


View matrix reprezentuje transformáciu z world space do camera space. V OpenGL pipeline:

View matrix vlastne hovorí: "Kde je kamera a kam sa pozerá?"

Application::Application()
{
    camera = new Camera(glm::vec3(0.0f, 0.3f, 2.0f));  // Dynamická kamera pre scénu 7

    // ✅ STATICKÁ view matrix - FIXED pohľad pre scény 1-6
    staticViewMatrix = glm::lookAt(
        glm::vec3(0.0f, 0.0f, 3.0f),   // Eye: kamera je 3 jednotky PRED obrazovkou (Z+)
        glm::vec3(0.0f, 0.0f, 0.0f),   // Center: pozerá sa na stred (origin)
        glm::vec3(0.0f, 1.0f, 0.0f)    // Up: Y os smeruje hore
    );
}

Čo znamená glm::lookAt(eye, center, up)?

Y (Up)
         ↑
         |
         |      Eye (0, 0, 3)
         |        👁️
         |       /
         |      / (pohľad)
         |     /
         |    ↙
    ─────┼─────────→ X
        /|    Center (0, 0, 0)
       / |
      /  |
     Z   |


Výsledok:

Kamera je na pozícii (0, 0, 3) - 3 jednotky pred scénou (ako divák v kine)
Pozerá sa priamo na stred (0, 0, 0)
Toto je ortogonálny/2D pohľad - ideálny pre ploché scény

Prečo je toto vhodné:

Scény 1-6 NIKDY nemenia pohľad → zbytočné vytvárať kamery
staticViewMatrix je len 64 bytes (4×4 float matrix)
Jednoduchšie - jedna "fotografia" pohľadu, ktorá sa len načíta
     
Metóda restoreStaticViewMatrix():

void Application::restoreStaticViewMatrix()
{
    // ✅ PREPÍŠ view matrix v GPU uniform premenných
    shaderProgram1->use();
    shaderProgram1->setUniform("viewMatrix", staticViewMatrix);  
    // ↑ OpenGL príkaz: glUniformMatrix4fv(..., staticViewMatrix)
    
    shaderProgram2->use();
    shaderProgram2->setUniform("viewMatrix", staticViewMatrix);
    
    shaderProgramTree->use();
    shaderProgramTree->setUniform("viewMatrix", staticViewMatrix);
    // ... atď pre všetky shadery ...

    printf("View matrix restored to static (2D scenes)\n");
}


```cpp

CPU Side (Aplikácia)                      GPU Side (Shader)
─────────────────────────────────────────────────────────────

staticViewMatrix (64 bytes)               uniform mat4 viewMatrix;
┌──────────────────┐                      ┌──────────────────┐
│ 1.0  0.0  0.0  0.0│                     │  ???  (nedef.)  │
│ 0.0  1.0  0.0  0.0│  ─────────────────>│                  │
│ 0.0  0.0  1.0 -3.0│  glUniformMatrix4fv │                  │
│ 0.0  0.0  0.0  1.0│                     │                  │
└──────────────────┘                      └──────────────────┘

Po volaní setUniform():
                                          ┌──────────────────┐
                                          │ 1.0  0.0  0.0  0.0│
                                          │ 0.0  1.0  0.0  0.0│
                                          │ 0.0  0.0  1.0 -3.0│
                                          │ 0.0  0.0  0.0  1.0│
                                          └──────────────────┘
                                          ✅ Shader teraz používa
                                             statický pohľad

```
Je to prepísanie GPU uniform premennej novými dátami
staticViewMatrix existuje celý čas v RAM, len sa nahrá do GPU

Pre scény 1-6:

Používame perspektívne premietanie (rovnaké ako scéna 7)
Statická view matrix je ďaleko od scény → efekt je "quasi-ortogonálny"
Objekty sú malé a ploché → perspektíva nie je výrazná

Záver: Postačovalo by ortogonálne, ale perspektívne funguje rovnako dobre a máme konzistenciu pre všetky scény.



------ ako pristupujeme ku samotným callbackom ? ----- 


Problém

// glfwSetKeyCallback očakáva STATICKÚ funkciu (nie member funkciu)
glfwSetKeyCallback(window, &Application::key_callback);  // ❌ NEFUNGUJE!


Prečo?

GLFW je C knižnica → očakáva statické C-style callbacky
Member funkcie majú skrytý this pointer → nie sú kompatibilné

Naše riešenie: Lambda wrapper + glfwSetWindowUserPointer
```cpp
void Application::initialization()
{
    // ... GLFW/GLEW init ...
    
    window = glfwCreateWindow(800, 600, "ZPG project", NULL, NULL);
    
    // ✅ KROK 1: Ulož THIS pointer do okna
    glfwSetWindowUserPointer(window, this);
    //                              ↑
    //                              Pointer na Application inštanciu
}
```

Čo sa stalo:

GLFW si zapamätá pointer na našu Application inštanciu
Môžeme ho neskôr vyžiadať späť v callbackoch


Krok 2: Lambda wrapper na konverziu statického callbacku

```cpp

cppvoid Application::initialization()
{
    // ✅ KROK 2: Lambda funkcia ako "bridge"
    glfwSetKeyCallback(window, [](GLFWwindow* window, int key, int scancode, 
                                   int action, int mods)
    {
        // ✅ KROK 3: Získaj THIS pointer z okna
        Application* app = static_cast<Application*>(
            glfwGetWindowUserPointer(window)
        );
        
        // ✅ KROK 4: Zavolaj member funkciu
        app->key_callback(window, key, scancode, action, mods);
        //  ↑ Teraz máme prístup k app->camera, app->sceneManager, atď.
    });
}
```
Rozpisujem to krok po kroku:
Fáza A: Registrácia (v initialization())
cpp// T0: Vytvorenie okna
window = glfwCreateWindow(...);

// T1: Uloženie this pointera
glfwSetWindowUserPointer(window, this);
//                              ↑
//                       this = adresa Application objektu (napr. 0x7fff1234)

// Interná štruktúra GLFW (zjednodušene):
struct GLFWwindow {
    void* userPointer;  // = 0x7fff1234 (náš Application*)
    // ... ostatné dáta ...
};
Fáza B: Callback trigger (keď hráč stlačí klávesu)
1. Hráč stlačí '7'
   ↓
2. GLFW detekuje input
   ↓
3. GLFW zavolá našu lambda funkciu:
   [](GLFWwindow* window, int key, int scancode, int action, int mods)
   ↓
4. Lambda získa THIS pointer:
   Application* app = static_cast<Application*>(glfwGetWindowUserPointer(window));
   //                                            ↑
   //                                    Vráti 0x7fff1234 (náš Application*)
   ↓
5. Lambda zavolá member funkciu:
   app->key_callback(window, key, scancode, action, mods);
   //  ↑ Teraz máme prístup k app->camera, app->sceneManager, atď!

Prístup ku kamere v callback metóde:

```cpp

cppvoid Application::key_callback(GLFWwindow* window, int key, int scancode, 
                               int action, int mods)
{
    // ✅ Priamy prístup k member premenným (lebo sme v member funkcii)
    
    if (action == GLFW_PRESS)
    {
        if (key >= GLFW_KEY_1 && key <= GLFW_KEY_7)
        {
            int sceneIndex = key - GLFW_KEY_1;
            
            // ✅ Prístup k sceneManager
            sceneManager->switchToScene(sceneIndex);
            
            if (sceneIndex == 6)  // Scéna 7
            {
                // ✅ Prístup k camera
                this->firstMouse = true;
                
                // ✅ Prístup k camera cez getViewMatrix()
                shaderProgram1->use();
                shaderProgram1->setUniform("viewMatrix", 
                                          this->camera->getViewMatrix());
                //                        ↑
                //                        Priamy prístup k member premennej
            }
            else
            {
                // ✅ Prístup k metódam
                this->restoreStaticViewMatrix();
            }
        }
    }
}
```
Dôležité:

this pointer je implicitný v member funkciách
Môžeme písať camera namiesto this->camera
Máme plný prístup k všetkým member premenným a metódam


Iné callbacky používajú rovnakú techniku:
```cpp
cppvoid Application::initialization()
{
    // ✅ Cursor position callback
    glfwSetCursorPosCallback(window, [](GLFWwindow* window, double xpos, double ypos)
    {
        Application* app = static_cast<Application*>(glfwGetWindowUserPointer(window));
        app->cursor_pos_callback(window, xpos, ypos);
        //  ↑ V tejto metóde môžeme pristupovať k app->camera
    });
    
    // ✅ Mouse button callback
    glfwSetMouseButtonCallback(window, [](GLFWwindow* window, int button, 
                                          int action, int mode)
    {
        Application* app = static_cast<Application*>(glfwGetWindowUserPointer(window));
        app->mouse_button_callback(window, button, action, mode);
    });
    
    // Všetky používajú ROVNAKÝ pattern!
}

Príklad: Cursor callback s prístupom ku kamere
cppvoid Application::cursor_pos_callback(GLFWwindow* window, double xpos, double ypos)
{
    // Ak NIE sme v scéne 7, použi starú funkcionalitu
    if (sceneManager->getActiveSceneIndex() != 6)
    {
        printf("cursor_pos_callback %d, %d; %d, %d\n", 
               (int)xpos, (int)ypos, (int)clickX, (int)clickY);
        return;
    }

    // ✅ Prístup k member premenným
    if (firstMouse)  // = this->firstMouse
    {
        lastX = xpos;  // = this->lastX
        lastY = ypos;  // = this->lastY
        firstMouse = false;
    }

    double xOffset = xpos - lastX;
    double yOffset = lastY - ypos;

    lastX = xpos;
    lastY = ypos;

    // ✅ Prístup ku kamere
    camera->processMouseMovement(xOffset, yOffset);
    //     ↑ = this->camera->processMouseMovement(...)
}

Vizualizácia celého flow:
┌─────────────────────────────────────────────────────────────┐
│ Application object (0x7fff1234)                             │
│                                                               │
│ ┌─────────────────┐                                         │
│ │ camera          │ ← Member premenná                       │
│ │ sceneManager    │                                         │
│ │ shaderProgram1  │                                         │
│ │ staticViewMatrix│                                         │
│ │ ...             │                                         │
│ └─────────────────┘                                         │
│                                                               │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ void key_callback(...)                                  │ │
│ │ {                                                       │ │
│ │     this->camera->getViewMatrix();  ← Priamy prístup   │ │
│ │     this->sceneManager->switchToScene(...);            │ │
│ │ }                                                       │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                    ↑
                    │ Prístup cez static_cast
                    │
┌───────────────────┴─────────────────────────────────────────┐
│ GLFW Lambda Wrapper (statická funkcia)                      │
│                                                               │
│ [](GLFWwindow* window, int key, ...)                        │
│ {                                                             │
│     Application* app = static_cast<Application*>(           │
│         glfwGetWindowUserPointer(window)                    │
│     );                                                        │
│     app->key_callback(window, key, ...);                    │
│ }                                                             │
└─────────────────────────────────────────────────────────────┘
                    ↑
                    │ GLFW zavolá pri inpute
                    │
┌───────────────────┴─────────────────────────────────────────┐
│ GLFW Internals                                               │
│                                                               │
│ struct GLFWwindow {                                          │
│     void* userPointer = 0x7fff1234;  ← Uložený this pointer│
│     callback_function = [lambda];     ← Registrovaný callback│
│ }                                                             │


```
Záver
Otázka 1:

NIE, používame perspektívne premietanie pre všetky scény
Scény 1-6 vyzerajú "ortogonálne", lebo kamera je ďaleko
Ortogonálne by stačilo, ale perspektívne je konzistentnejšie

Otázka 2:

Používame lambda wrapper + glfwSetWindowUserPointer
GLFW si pamätá this pointer → lambda ho získa späť → zavolá member funkciu
V member funkcii máme priamy prístup k camera, sceneManager, atď.

Pattern summary:
cpp// Setup:
glfwSetWindowUserPointer(window, this);

// Callback:
glfwSetKeyCallback(window, [](GLFWwindow* w, ...) {
    Application* app = static_cast<Application*>(glfwGetWindowUserPointer(w));
    app->member_function(...);  // ← Teraz máme prístup ku všetkému
});
Toto je štandardný C++ pattern pre prácu s C knižnicami, ktoré vyžadujú statické callbacky!

DOTAZ : 

Kontext a naše riešenie:
V našom projekte máme 7 scén s rôznymi požiadavkami na kameru:

Scény 1-6: Statický 2D pohľad (objekty sa transformujú, ale pohľad sa nemení)
Scéna 7: Dynamická 3D FPS kamera (hráč sa pohybuje pomocou WSAD + myš)

Naše implementované riešenie:
Rozhodli sme sa pre hybridný prístup s jednou inštanciou kamery a statickou view matrix:

Naša obhajoba tohto riešenia:
1. Pamäťová efektivita:

Naše riešenie: ~264 bytes (1× Camera + 1× mat4)
Alternatíva (7× Camera): ~1400 bytes (7× Camera objektov)
Úspora: ~1136 bytes (81% redukcia)

. Princíp "Don't pay for what you don't use":

Scény 1-6 nepotrebujú dynamickú kameru → nedostanú plnohodnotný Camera objekt
Stačí im readonly view matrix → dostanú len 64-bytovú maticu
Observer pattern sa aktivuje len v scéne 7, kde je potrebný

Výhoda: Pozícia kamery "prežije" prepínanie medzi scénami - hráč sa vráti presne tam, kde bol.


Ak by sme pridali 50 statických scén a 5 dynamických:

Potenciálne nevýhody nášho riešenia (a prečo sme ich akceptovali):
Nevýhoda 1: Manuálne prepínanie view matrix


// Pri každom prepnutí scény musíme manuálne volať:
if (sceneIndex == 6)
    shaderProgram->setUniform("viewMatrix", camera->getViewMatrix());
else
    restoreStaticViewMatrix();)


    Prečo je to OK:

Deje sa len pri prepnutí scény (používateľský input) → nie každý frame
Jednoduché na debugging (vidíme explicitne, čo sa deje)
Nie je to performance bottleneck (pár uniform uploadov)

Nevýhoda 2: Duplicitný kód pre všetky shadery


Prečo je to OK:

Mohli by sme to refaktorovať do loop cez std::vector<ShaderProgram*>
Ale explicitný kód je čitateľnejší a jednoduchší na debug
Volá sa len pri prepnutí scény (nie performance-critical)

Alternatívne riešenia, ktoré sme NEPOUŽILI:
Alternatíva 1: Kamera pre každú scénu ❌
cppCamera* camera1;  // Scéna 1
Camera* camera2;  // Scéna 2
// ... atď.
Prečo sme to odmietli:

❌ 5× viac pamäte
❌ Zbytočná komplexita pre statické scény
❌ Observer pattern by sa aktivoval zbytočne pre scény 1-6

Alternatíva 2: Globálna view matrix premenná ❌
cppglm::mat4 g_viewMatrix;  // Globálny state

// V renderovaní:
shaderProgram->setUniform("viewMatrix", g_viewMatrix);
Prečo sme to odmietli:

❌ Globálny state (anti-pattern)
❌ Thread-unsafe
❌ Ťažké na testovanie
❌ Ťažšie na reasoning (kto mení g_viewMatrix?)

Alternatíva 3: Scene vlastní svoju view matrix ❌
cppclass Scene
{
    glm::mat4 viewMatrix;  // Každá scéna má vlastnú
};
Prečo sme to odmietli:

❌ Scéna 7 potrebuje dynamickú view matrix → Scene by musel vlastniť Camera
❌ Porušuje Single Responsibility Principle (Scene by robila rendering + camera management)
❌ Ťažšie zdieľať kameru medzi scénami


DOTAZ 



Naša otázka pre cvičiaceho:

Je náš prístup k správe kamery vhodný?
Obhajujeme toto riešenie:

1× Camera objekt pre dynamickú scénu 7
1× staticViewMatrix pre statické scény 1-6
Manuálne prepínanie view matrix pri zmene scény

Argumenty PRE:

Pamäťová efektivita: ~264 bytes vs. ~1400 bytes (81% úspora)
"Don't pay for what you don't use": Statické scény nedostanú zbytočnú funkcionalitu
Zachovanie stavu: Pozícia kamery prežije prepínanie scén
Škálovateľnosť: Pri 50 statických + 5 dynamických scénach úspora 91%

Argumenty PROTI:

Manuálne prepínanie view matrix (duplicitný kód)
Explicitné volanie restoreStaticViewMatrix() / camera->getViewMatrix()

Otázky:

Súhlasíte s našou voľbou? Aké sú z vášho pohľadu hlavné výhody/nevýhody?
Vidíte lepšie alternatívne riešenie, ktoré sme prehliadli?
Ak by sme mali škálovať na 100+ scén (mix statických a dynamických), čo by ste odporučili zmeniť?
Je rozumné používať perspektívne premietanie pre scény 1-6 (aj keď vyzerajú 2D), alebo by bolo lepšie prepnúť na ortogonálne pre statické scény?


plynutly prechod medzi scenami camera interpolacia 

použiť  Scene Management Pattern pre pridávanie scém 

Výhody Scene Management Pattern:
✅ Separácia zodpovedností: Každá scéna vie len o sebe
✅ Modulárnosť: Ľahko pridať novú scénu (nový .cpp súbor)
✅ Testovateľnosť: Môžeme testovať ForestScene izolované
✅ Škálovateľnosť: 100 scén = 100 malých tried, nie 1 obrovská metóda
✅ Lifecycle hooks: onEnter() / onExit() pre setup/cleanup
✅ Update logika: Každá scéna môže mať vlastnú update loop


2. Strategy Pattern
Čo to je?
Strategy Pattern je behaviorálny vzor, ktorý zapúzdruje algoritmy do samostatných tried a umožňuje ich dynamicky meniť za behu.
V našom prípade: StaticViewStrategy vs. DynamicViewStrategy

čo to u nas rieši : 


Problémy:

❌ IF-statement (conditional logic) v key_callback
❌ Application musí vedieť, ktorá scéna potrebuje akú view stratégiu
❌ Ťažké pridať tretiu stratégiu (napr. OrbitViewStrategy pre editor)


Detekcia kolizie s telesom 

A) Bounding Sphere (guľa) 
```plain
●  ← Center (x, y, z)
       ╱│╲
      ╱ │ ╲  ← Radius
     ●──┼──●
      ╲ │ ╱
       ╲│╱
        ●
```        
B) AABB (Axis-Aligned Bounding Box) - Kocka zarovnaná s osami
```plain
Y
      ↑
      │   max (x_max, y_max, z_max)
      │    ┌────────────┐
      │    │            │
      │    │  LAVIČKA   │  ← AABB
      │    │            │
      │    └────────────┘
      │   min (x_min, y_min, z_min)
      └─────────────────→ X
     ╱
    Z

```
```plain
C) OBB (Oriented Bounding Box) - Rotovaná kocka

        Y
      ↑       ╱────╲
      │      ╱      ╲  ← Rotovaný box
      │     │ LAVIČKA│
      │      ╲      ╱
      │       ╲────╱
      └─────────────────→ X
     ╱
    Z
```
```plain
 D) Mesh Collision (per-triangle)

Testovanie každého trojuholníka modelu
    ┌───┐
    │╲  │  ← 108 vertexov = 36 trojuholníkov
    │ ╲ │     = 36 testov!
    │  ╲│
    └───┘
   ```


ako funguje toto ? 

void Camera::notify()
{
    // 1. PREPOČÍTAJ view matrix (kamera sa pohla/otočila)
    viewMatrix = glm::lookAt(position, position + front, up);

    // pre kazdy ShaderProgram zavolaj jeho metodu update
    for (ICameraObserver* observer : observers)
    {                                   // pouzivame pull pattern
        observer->update(this); // Tu máš kameru, vytiahni si z nej čo potrebuješ
    }
}


Analógia: Pizzeria 🍕
Bez this (zlé):
Kamera (ty): "Hej ShaderProgram, pohol som sa, aktualizuj sa!"
ShaderProgram: "OK, ale... kde nájdem tvoje dáta??" 🤷‍♂️
S this (dobré):
Kamera (ty): "Hej ShaderProgram, pohol som sa! Tu ma máš (this), 
              pozri sa na moju pozíciu a view matrix!"
ShaderProgram: "Super, vidím ťa, vytiahnem si view matrix!" 👍

Finálne zhrnutie - NAJJEDNODUCHŠIE
cppfor (ICameraObserver* observer : observers)
{
    observer->update(this);
}
Pre každého v zozname:

observer = momentálny shader program
->update() = zavolaj funkciu update() na ňom
this = pošli mu seba (kameru), aby vedel, kto volá a odkiaľ si má vytiahnuť dáta
V jednej vete:
"Pre každý shader program v zozname zavolaj jeho funkciu update() a daj mu seba (kameru) ako parameter."


# Práca na parsery 

GPS koordináty a validácia 

# Tracker môže poslať chybné dáta:
latitude = 999.123456   # ❌ Neplatné (max je 90)
longitude = -200.5      # ❌ Neplatné (min je -180)

Kedy to môže nastať : tracker je v tunely / budove = nema gps signal, pošle default hodnotu - 0.0 0.0 niekde v atlantickom ocenae alebo garbage hodnoty 999  , -999 
alebo nijaký firmware bug, bit flip ale to je vysoko nepravdepodobný case 

= zabranujeme uloženiu nezmyslov, ochrana front endu, detekujeme hardware a firmware problemy 
= o vysledku mame teda data čiste pre samotnú analyzu 

--

battery validácia = chybne čitanie senzorov, firmware bugs
detekujeme : hardware problemy 
analytics : battery life analýza je presna 
UX : ukažem radšej unknown na frontende než 150 percent 

timezone-aware-datetime 
fix pre spravne ukladanie postgresSQL aby správne ukladala timestamp with timezone 


RSSI / SNR VAlidácia 

RSSI = meria kvalitu signálu v dBm ( decibel-milliwats)
rozsah od -120dBm velmi slaby až 0 dBM (perfektný)

SNR (signal to noise ratio)

Statistic tracking 

Monitorovanie zdravia systému 
detekcia firmware problemov 
optimalizácia trackera 
= z toho sa bude dať reportovať pre BP 

----------


 Composite Pattern - Detailná analýza

### 📊 Štruktúra UML diagramu
```text

drawableobject(client)   --->    TransformComponent (abstraktná)
                            |
                            |
           +----------------+----------------+
          |                                 |
    LEAF triedy                    TransformComposite
    (elementárne)                       |
          |                             |
    - Translate                   vector<TransformComponent*>
    - Rotate                            |
    - Scale                             | môže obsahovať
                                        v
                              - Translate/Rotate/Scale
                              - ALEBO ďalší TransformComposite!


```
Vysvetlenie : 

DrawableObject je klient, ktorý používa TransformComponent*
Nezaujíma ho, či dostane Leaf alebo Composite
Presne ako na UML diagrame: Client → Component

Component → TransformComponent





// ✅ JASNÉ názvy:
TransformComponent    // "Aha, toto je Component z Composite pattern"
TransformComposite    // "Aha, toto je Composite z Composite pattern"
Translate, Rotate     // "Aha, toto sú Leaf elementy"


// Vytvor "sub-group" transformácií
TransformComposite* subGroup = new TransformComposite();
subGroup->addComponent(new Translate(1.0f, 0.0f, 0.0f));
subGroup->addComponent(new Scale(0.5f, 0.5f, 0.5f));

// Hlavný composite obsahuje sub-group!
TransformComposite* main = new TransformComposite();
main->addComponent(subGroup);  // ✅ Composite obsahuje Composite!
main->addComponent(new Rotate(90.0f, 0.0f, 1.0f, 0.0f));

DrawableObject* obj = new DrawableObject(model, main, shader);
// ✅ Rekurzia! getMatrix() sa zavolá rekurzívne


=== prakticky nám to umožní dať lubovolny počet transformcomponent teda vytvoriť tzv zloženú transformáciu z elementárnych 

implementácia observera 

Prečo je pull pattern ten správny prístup ?

Subject nerozhoduje čo poslať, len hovorí "hej" niečo sa zmenilo 
Každý observer si vezme len to, čo potrebuje
Pridanie nového observera nevyžaduje zmenu interface


```cpp

// SPRÁVNE - to čo máte:
class ICameraObserver {
    virtual void update(Camera* camera) = 0;
    //                  ^^^^^^^^^^^^^^
    //                  Observer si vytiahne ČO potrebuje
};

// ShaderProgram - potrebuje view matrix
void ShaderProgram::update(Camera* camera) {
    setUniform("viewMatrix", camera->getViewMatrix());
}

// Controller - potrebuje len position (hypoteticky)
void Controller::update(Camera* camera) {
    glm::vec3 pos = camera->getPosition();
    // Spracuj len pozíciu
}

// Light - potrebuje direction (hypoteticky)
void Light::update(Camera* camera) {
    glm::vec3 dir = camera->getFront();
    // Spracuj len smer
}
```

ako funguje scéna so slnečnou sústavou ? 

glm::mat4 getMatrix() {
    glm::mat4 result = glm::mat4(1.0f);  // I (jednotková matica)
    
    // Transformácia 1: Rotate(earthAngle)
    result = result * transformations[0]->getMatrix();
    // result = I * R1
    
    // Transformácia 2: Translate(1.2, 0, 0)
    result = result * transformations[1]->getMatrix();
    // result = I * R1 * T1
    
    // Transformácia 3: Rotate(moonAngle)
    result = result * transformations[2]->getMatrix();
    // result = I * R1 * T1 * R2
    
    // Transformácia 4: Translate(0.4, 0, 0)
    result = result * transformations[3]->getMatrix();
    // result = I * R1 * T1 * R2 * T2
    
    // Transformácia 5: Scale(0.05)
    result = result * transformations[4]->getMatrix();
    // result = I * R1 * T1 * R2 * T2 * S
    
    return result;
}
```

### **Finálna matica:**
```
Mesiac = R1(earthAngle) × T1(1.2, 0, 0) × R2(moonAngle) × T2(0.4, 0, 0) × S(0.05)
         ↑               ↑                ↑                ↑                ↑
         Rotuj okolo     Choď k Zemi      Rotuj okolo     Vzdialenosť     Veľkosť
         Slnka                            Zeme            od Zeme         Mesiaca
         
## ✅ Výhody Composite Pattern

### **1. Hierarchické transformácie**
```
Slnko (0, 0, 0)
  │
  └─ Zem (rotuje okolo Slnka)
       │
       └─ Mesiac (rotuje okolo Zeme, ktorá rotuje okolo Slnka)
         
// ✅ Jednoducho pridáš transformácie v poradí
moonOrbit->addTransformation(new Rotate(earthAngle, 0.0f, 1.0f, 0.0f));
moonOrbit->addTransformation(new Translate(1.2f, 0.0f, 0.0f));
moonOrbit->addTransformation(new Rotate(moonAngle, 0.0f, 1.0f, 0.0f));
moonOrbit->addTransformation(new Translate(0.4f, 0.0f, 0.0f));

// ✅ Composite sa postará o násobenie matíc!


// # Strategy pattern pre načítanie z filu opýtaj sa na cvikách 

Dobrý deň,
Chcel by som sa opýtať na implementáciu Strategy Patternu pri načítavaní shaderov.
Vytvoril som si interface IShaderLoader s dvoma implementáciami:

FileShaderLoader - načítava shader kód zo súboru (.vert, .frag)
InlineShaderLoader - umožňuje vložiť shader kód priamo ako string

Dôvod: V budúcnosti by som mohol pridať ďalšie stratégie, napríklad:

NetworkShaderLoader - sťahovanie shaderov zo servera
CompressedShaderLoader - načítanie zo ZIP archívu
DatabaseShaderLoader - uložené v databáze

Je tento prístup správny z pohľadu návrhu? Alebo je lepšie použiť len jednoduchú funkciu na načítanie zo súboru bez pattern?
Ďakujem za odpoveď.

Bonus: Argumenty PRE Strategy Pattern
Ak ti učiteľ povie "Prečo to nemáš len ako funkciu?", použi tieto argumenty:

Open/Closed Principle - Môžem pridať novú stratégiu bez úpravy Shader triedy
Testovanie - Jednoducho si vytvorím MockShaderLoader pre unit testy
Flexibilita - V budúcnosti:

NetworkShaderLoader - hot-reload shaderov z dev servera
CachedShaderLoader - cache skompilovaných shaderov
EncryptedShaderLoader - šifrované shadery (ochrana IP)


Separation of Concerns - Shader trieda nemusí vedieť o file I/


# Podstatné 

### **Dôležité pochopenie:**
```
Subject (Light, Camera)  ----notifikuje---->  Observer (ShaderProgram)
   ^                                              ^
   |                                              |
Dedí: Light, Camera                      Dedí: ShaderProgram

Prečo ShaderProgram dedí z Observer?

Pretože ShaderProgram POČÚVA zmeny (Light a Camera ho notifikujú):


// V Application.cpp:
Light* light = new Light();
Camera* camera = new Camera();
ShaderProgram* shader = new ShaderProgram();

// ShaderProgram sa registruje ako observer:
light->attach(shader);    // Light má attach() zo Subject
camera->attach(shader);   // Camera má attach() zo Subject

// Keď sa Light zmení:
light->setPosition(newPos);
light->notifyAll();  // Zavolá shader->notify(light) ← TU SA VOLÁ!

# FORWARD DEKLÁRÁCIA

 Forward Deklarácia: class Observer;Čo to znamená?Forward deklarácia hovorí kompilátoru: "Trieda Observer existuje, ale jej definíciu uvidíš neskôr."

## Prečo potrebujeme forward deklaráciu?
Problém: Circular Dependency (Kruhová závislosť)
Bez forward deklarácie by nastal problém:

// ❌ BEZ FORWARD DEKLARÁCIE:

// Subject.h
#include "Observer.h"  // ← Subject potrebuje Observer

class Subject {
    vector<Observer*> observers;
};


// Observer.h
#include "Subject.h"   // ← Observer potrebuje Subject

class Observer {
    virtual void notify(Subject* s) = 0;
};


// 💥 PROBLÉM:
Subject.h → include Observer.h → include Subject.h → include Observer.h → ...
                    ∞ NEKONEČNÁ SLUČKA!


📋 Kedy použiť Forward Deklaráciu?
SituáciaForward Deklarácia?DôvodPointer na triedu✅ ÁNOKompilátor nepotrebuje veľkosť objektuReferencia na triedu✅ ÁNOKompilátor nepotrebuje veľkosť objektuMetóda vracia triedu✅ ÁNO (v .h)Len deklarácia, definícia v .cppAtribút je objekt❌ NIEKompilátor potrebuje veľkosť objektuVolanie metód objektu❌ NIEKompilátor potrebuje plnú definíciu
                    
Impleentácia observera u nás 

2️⃣ LIGHT a CAMERA dedia zo SUBJECT


Prečo je to čisto virtuálna metóda?

✅ Je to interface (kontrakt)
✅ Každý observer MUSÍ implementovať notify()
✅ Každý observer rozhodne sám, AKO reaguje

// Observer.h
class Observer {
public:
    virtual void notify(Subject* subject) = 0;  // ← Čisto virtuálna!
    //                   ^^^^^^
    //                   Kto ma notifikoval?
};

# Aký je vzťah Observer ↔ Subject?

OBOJSMERNÁ ZÁVISLOSŤ (Bidirectional Dependency)

Čo znamená virtual?

Metóda bude polymorfná (môže mať rôzne implementácie)
C++ sa rozhodne za behu (runtime), ktorú konkrétnu implementáciu zavolať
Toto sa volá dynamic dispatch alebo late binding

// Subject.cpp
void Subject::notifyAll() {
    for (Observer* observer : observerCollection) {
        //   ^^^^^^^^
        //   Toto je TYP: Observer*
        //   Ale v skutočnosti ukazuje na ShaderProgram objekt!
        
        if (observer) {
            observer->notify(this);  // ← KÚZLO sa deje TU!
            //       ^^^^^^
            //       Ktorú notify() zavolať?
        }
    }
}
```

**Čo sa stane keď zavoláme `observer->notify(this)`?**
```
┌──────────────────────────────────────────────────────────┐
│  observer->notify(this)                                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. observer je typu Observer* (základný typ)            │
│  2. ALE v skutočnosti ukazuje na ShaderProgram objekt    │
│  3. notify() je VIRTUÁLNA metóda                         │
│  4. C++ za behu zistí: "Aha, tento observer je           │
│     v skutočnosti ShaderProgram!"                        │
│  5. Zavolá ShaderProgram::notify() ✅                    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 🎨 Vizualizácia - VIRTUÁLNA TABUĽKA (vtable)

Každý objekt, ktorý má virtuálne metódy, má vnútorne skrytú **vtable** (virtual table):
```
OBSERVER (abstraktná trieda)
═══════════════════════════════════════════
┌─────────────────────┐
│ Observer            │
│─────────────────────│
│ vtable* ───────────►│ notify() = 0 (čisto virtuálna)
└─────────────────────┘


SHADERPROGRAM (konkrétna implementácia)
═══════════════════════════════════════════
┌─────────────────────────────┐
│ ShaderProgram               │
│─────────────────────────────│
│ vtable* ──────────────►┌────────────────────────┐
│                        │ notify() ────► ShaderProgram::notify()
│ - uniformLocations     └────────────────────────┘
│ - programID
└─────────────────────────────┘


KEĎ VOLÁME observer->notify(this):
═══════════════════════════════════════════

observer je typu Observer*, ale ukazuje na ShaderProgram objekt

┌────────────────────┐
│ observer (Observer*)│
└──────────┬─────────┘
           │
           │ ukazuje na
           ↓
┌─────────────────────────────┐
│ ShaderProgram objekt        │
│─────────────────────────────│
│ vtable* ──────────────►┌────────────────────────┐
│                        │ notify() ────► ShaderProgram::notify()
│                        └────────────────────────┘
└─────────────────────────────┘
                                   ↑
                                   │
                    C++ zavolá túto implementáciu!




// ═══════════════════════════════════════════════════════
// KROK 1: Vytvorenie objektov (Application.cpp)
// ═══════════════════════════════════════════════════════

1. ShaderProgram* shader = new ShaderProgram();
   // Vytvorí sa ShaderProgram objekt s vtable obsahujúcou
   // ShaderProgram::notify()

2. Camera* camera = new Camera();
   // Vytvorí sa Camera objekt (má observerCollection)

3. camera->attach(shader);
   // Subject.cpp:
   void Subject::attach(Observer* observer) {
       //               ^^^^^^^^ 
       //               shader sa pretypuje na Observer*
       observerCollection.push_back(observer);
   }
   
   // observerCollection teraz obsahuje:
   // [0] ───► Observer* (skutočne ukazuje na ShaderProgram!)


// ═══════════════════════════════════════════════════════
// KROK 2: Zmena v Camera (Camera.cpp)
// ═══════════════════════════════════════════════════════

4. camera->moveForward(0.1f);
   void Camera::moveForward(float dt) {
       position += forward * speed * dt;
       notifyAll();  // ← Volám metódu zo Subject
   }


// ═══════════════════════════════════════════════════════
// KROK 3: notifyAll() iteruje cez observerov (Subject.cpp)
// ═══════════════════════════════════════════════════════

5. void Subject::notifyAll() {
       for (Observer* observer : observerCollection) {
           //  ^^^^^^^^
           //  observer má typ Observer*
           //  ale v skutočnosti ukazuje na ShaderProgram!
           
           if (observer) {
               observer->notify(this);  // ← RIADOK 6
               //       ^^^^^^
               //       Ktorú notify() zavolať?
               //       
               //       C++ za behu pozrie do vtable:
               //       "Tento observer je ShaderProgram,
               //        takže zavolám ShaderProgram::notify()!"
           }
       }
   }


// ═══════════════════════════════════════════════════════
// KROK 4: ShaderProgram::notify() sa vykoná! (ShaderProgram.cpp)
// ═══════════════════════════════════════════════════════

6. void ShaderProgram::notify(Subject* subject) {
       //                      ^^^^^^^ 
       //                      subject = Camera*
       
       Camera* camera = dynamic_cast<Camera*>(subject);
       if (camera) {
           updateCameraUniforms(camera);  // ✅ Vykoná sa!
       }
   }
```

---

## 💡 Kľúčové pochopenie
```
┌────────────────────────────────────────────────────────┐
│  AKO observer->notify() VIE O SHADERPROGRAME?         │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. notify() je VIRTUÁLNA metóda v Observer           │
│  2. ShaderProgram OVERRIDUJE notify()                 │
│  3. observerCollection obsahuje Observer* ukazovateľ  │
│     ktorý v skutočnosti ukazuje na ShaderProgram      │
│  4. Keď voláme observer->notify(), C++ za behu        │
│     (runtime) pozrie do vtable a zistí:               │
│     "Tento observer je ShaderProgram, zavolám         │
│     ShaderProgram::notify()!"                         │
│                                                        │
│  Toto sa volá POLYMORFIZMUS! 🎯                       │
└────────────────────────────────────────────────────────┘



## Lopaticke vysvetlenie observera 


// Application.cpp
ShaderProgram* shader = new ShaderProgram();  // ← Vytvorím ShaderProgram
Camera* camera = new Camera();                 // ← Vytvorím Camera

camera->attach(shader);  // ← ShaderProgram sa stane OBSERVEROM kamery!
//             ^^^^^^
//             shader je UKAZOVATEĽ na ShaderProgram
```

---

## 📊 Vizualizácia toho, čo sa deje

### **KROK 1: Vytvorenie objektov v pamäti**
```
HEAP PAMÄŤ:
═══════════════════════════════════════════════════

┌─────────────────────────────────────┐
│ ShaderProgram objekt                │  ◄─── shader ukazuje sem
│ Adresa: 0x1000                      │
│─────────────────────────────────────│
│ vtable* ──►[notify() = ShaderProgram::notify()]
│ programID: 5                        │
│ uniformLocations: {...}             │
└─────────────────────────────────────┘

shader je UKAZOVATEĽ (pointer)
     │
     │ Hodnota: 0x1000
     └─► Ukazuje na ShaderProgram objekt


┌─────────────────────────────────────┐
│ Camera objekt                       │  ◄─── camera ukazuje sem
│ Adresa: 0x2000                      │
│─────────────────────────────────────│
│ observerCollection: []   ← Prázdny vector (zatiaľ)
│ position: {0, 0, 0}                 │
│ forward: {0, 0, -1}                 │
└─────────────────────────────────────┘

camera je UKAZOVATEĽ (pointer)
     │
     │ Hodnota: 0x2000
     └─► Ukazuje na Camera objekt


camera->attach(shader);
//      ^^^^^^ ^^^^^^
//        │      │
//        │      └─ ShaderProgram* (typ: ShaderProgram*)
//        │         Hodnota: 0x1000
//        │
//        └─ Metóda zo Subject


// Subject.cpp
void Subject::attach(Observer* observer) {
    //                 ^^^^^^^^ 
    //                 Parameter má TYP: Observer*
    //                 ALE DOSTANE: ShaderProgram* (0x1000)
    //                 
    //                 IMPLICITNÉ PRETYPOVANIE:
    //                 ShaderProgram* → Observer* (upcasting)
    //                 ✅ Bezpečné, pretože ShaderProgram dedí z Observer!
    
    if (observer != nullptr) {
        observerCollection.push_back(observer);
        //                           ^^^^^^^^
        //                           Uloží sa Observer* (0x1000)
    }
}
```

---

### **KROK 3: Stav po attach()**
```
┌─────────────────────────────────────────────────────────┐
│ Camera objekt (0x2000)                                  │
│─────────────────────────────────────────────────────────│
│ observerCollection:                                     │
│   [0] ───► Observer* (0x1000)  ◄─┐                     │
│                                   │                     │
│ position: {0, 0, 0}               │                     │
│ forward: {0, 0, -1}               │                     │
└───────────────────────────────────┼─────────────────────┘
                                    │
                                    │ Ukazuje na...
                                    │
                ┌───────────────────┘
                │
                ↓
┌─────────────────────────────────────┐
│ ShaderProgram objekt (0x1000)       │
│─────────────────────────────────────│
│ vtable* ──►[notify() = ShaderProgram::notify()]
│ programID: 5                        │
│ uniformLocations: {...}             │
└─────────────────────────────────────┘

          ↑
          │
shader ───┘ (stále ukazuje na rovnaký objekt!)


void Subject::notifyAll() {
    for (Observer* observer : observerCollection) {
        //  ^^^^^^^^
        //  Typ: Observer*
        //  Skutočný objekt: ShaderProgram
        
        observer->notify(this);  // ← TU je RUNTIME polymorfizmus!
        //       ^^^^^^
        //       C++ za behu (runtime) zistí:
        //       "Tento observer je ShaderProgram,
        //        zavolám ShaderProgram::notify()!"
    }




void Subject::notifyAll() {
    for (Observer* observer : observerCollection) {
        //  ^^^^^^^^
        //  Typ: Observer*
        //  Skutočný objekt: ShaderProgram
        
        observer->notify(this);  // ← TU je RUNTIME polymorfizmus!
        //       ^^^^^^
        //       C++ za behu (runtime) zistí:
        //       "Tento observer je ShaderProgram,
        //        zavolám ShaderProgram::notify()!"
    }
}
```

---

## 📋 Detailná tabuľka typov a hodnôt

| Premenná | Typ | Hodnota (príklad) | Čo je to? |
|----------|-----|-------------------|-----------|
| `shader` | `ShaderProgram*` | `0x1000` | Ukazovateľ na ShaderProgram objekt |
| `camera` | `Camera*` | `0x2000` | Ukazovateľ na Camera objekt |
| `observerCollection[0]` | `Observer*` | `0x1000` | Ukazovateľ na Observer (ale skutočne na ShaderProgram!) |

---

## 🎨 Vizualizácia pretypovaní
```
COMPILE-TIME (pri kompilácii):
═══════════════════════════════════════════════════

camera->attach(shader);
//             ^^^^^^
//             Typ: ShaderProgram* (0x1000)

       │
       │ IMPLICITNÉ PRETYPOVANIE (upcasting)
       │ ShaderProgram* ──► Observer*
       ↓

void Subject::attach(Observer* observer)
//                    ^^^^^^^^
//                    Typ: Observer* (0x1000)
//                    
//                    STÁLE ukazuje na ROVNAKÝ objekt!
//                    Len typ sa zmenil:
//                    ShaderProgram* ──► Observer*


RUNTIME (počas behu programu):
═══════════════════════════════════════════════════

observer->notify(this);
//       ^^^^^^

C++ pozrie do vtable objektu na adrese 0x1000:
┌─────────────────────────────────────┐
│ Objekt na 0x1000:                   │
│ Typ: ShaderProgram                  │
│ vtable ──►[notify() = ShaderProgram::notify()]
└─────────────────────────────────────┘
              │
              │ ZAVOLÁ TÚ METÓDU!
              ↓
void ShaderProgram::notify(Subject* subject) {
    // Táto metóda sa vykoná!
}


// ═══════════════════════════════════════════════════════
// KROK 1: Vytvorenie objektov
// ═══════════════════════════════════════════════════════

ShaderProgram* shader = new ShaderProgram();
// ✅ shader je UKAZOVATEĽ
// ✅ Hodnota: 0x1000 (príklad)
// ✅ Ukazuje na ShaderProgram objekt v heap pamäti

Camera* camera = new Camera();
// ✅ camera je UKAZOVATEĽ
// ✅ Hodnota: 0x2000 (príklad)
// ✅ Ukazuje na Camera objekt v heap pamäti
// ✅ Camera má prázdny observerCollection


// ═══════════════════════════════════════════════════════
// KROK 2: Registrácia observera
// ═══════════════════════════════════════════════════════

camera->attach(shader);
//      ^^^^^^ ^^^^^^
//        │      └─ ShaderProgram* (0x1000)
//        │         COMPILE-TIME: pretypuje sa na Observer*
//        │
//        └─ Metóda zo Subject

// Vnútri attach():
void Subject::attach(Observer* observer) {
    // observer = Observer* (0x1000)
    // Stále ukazuje na ShaderProgram objekt!
    
    observerCollection.push_back(observer);
    // Vector teraz obsahuje:
    // [0] ───► Observer* (0x1000)
    //          (skutočne ShaderProgram objekt)
}


// ═══════════════════════════════════════════════════════
// KROK 3: Notifikácia
// ═══════════════════════════════════════════════════════

camera->moveForward(0.1f);
// ──► Camera::moveForward() zavolá notifyAll()

void Subject::notifyAll() {
    for (Observer* observer : observerCollection) {
        // observer = Observer* (0x1000)
        // Typ: Observer*
        // Skutočný objekt: ShaderProgram
        
        observer->notify(this);  // ← RUNTIME POLYMORFIZMUS!
        // C++ za behu pozrie do vtable na adrese 0x1000
        // Zistí: "Toto je ShaderProgram!"
        // Zavolá: ShaderProgram::notify(camera)
    }
}


// ═══════════════════════════════════════════════════════
// KROK 4: ShaderProgram reaguje
// ═══════════════════════════════════════════════════════

void ShaderProgram::notify(Subject* subject) {
    // subject = Camera* (0x2000)
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    if (camera) {  // ✅ TRUE
        updateCameraUniforms(camera);
    }
}
```

---

## 🏆 Zhrnutie
```
┌────────────────────────────────────────────────────────┐
│  TVOJE POCHOPENIE:                                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ✅ shader je UKAZOVATEĽ na ShaderProgram             │
│  ✅ shader sa stáva OBSERVEROM kamery                 │
│  ✅ shader je PRVOK v observerCollection              │
│  ✅ shader je pretypovaný na Observer*                │
│     (compile-time upcasting)                          │
│  ✅ C++ za behu (runtime) vie, že je to ShaderProgram │
│     a zavolá správnu notify() metódu                  │
│                                                        │
│  JE TO 100% SPRÁVNE! 🎯                               │
└────────────────────────────────────────────────────────┘


// ═══════════════════════════════════════════════════════
// KROK 1: Vytvorenie objektov
// ═══════════════════════════════════════════════════════

ShaderProgram* shader = new ShaderProgram();
// ✅ shader je UKAZOVATEĽ
// ✅ Hodnota: 0x1000 (príklad)
// ✅ Ukazuje na ShaderProgram objekt v heap pamäti

Camera* camera = new Camera();
// ✅ camera je UKAZOVATEĽ
// ✅ Hodnota: 0x2000 (príklad)
// ✅ Ukazuje na Camera objekt v heap pamäti
// ✅ Camera má prázdny observerCollection


// ═══════════════════════════════════════════════════════
// KROK 2: Registrácia observera
// ═══════════════════════════════════════════════════════

camera->attach(shader);
//      ^^^^^^ ^^^^^^
//        │      └─ ShaderProgram* (0x1000)
//        │         COMPILE-TIME: pretypuje sa na Observer*
//        │
//        └─ Metóda zo Subject

// Vnútri attach():
void Subject::attach(Observer* observer) {
    // observer = Observer* (0x1000)
    // Stále ukazuje na ShaderProgram objekt!
    
    observerCollection.push_back(observer);
    // Vector teraz obsahuje:
    // [0] ───► Observer* (0x1000)
    //          (skutočne ShaderProgram objekt)
}


// ═══════════════════════════════════════════════════════
// KROK 3: Notifikácia
// ═══════════════════════════════════════════════════════

camera->moveForward(0.1f);
// ──► Camera::moveForward() zavolá notifyAll()

void Subject::notifyAll() {
    for (Observer* observer : observerCollection) {
        // observer = Observer* (0x1000)
        // Typ: Observer*
        // Skutočný objekt: ShaderProgram
        
        observer->notify(this);  // ← RUNTIME POLYMORFIZMUS!
        // C++ za behu pozrie do vtable na adrese 0x1000
        // Zistí: "Toto je ShaderProgram!"
        // Zavolá: ShaderProgram::notify(camera)
    }
}


// ═══════════════════════════════════════════════════════
// KROK 4: ShaderProgram reaguje
// ═══════════════════════════════════════════════════════

void ShaderProgram::notify(Subject* subject) {
    // subject = Camera* (0x2000)
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    if (camera) {  // ✅ TRUE
        updateCameraUniforms(camera);
    }
}
```

---

## 🏆 Zhrnutie
```
┌────────────────────────────────────────────────────────┐
│  TVOJE POCHOPENIE:                                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ✅ shader je UKAZOVATEĽ na ShaderProgram             │
│  ✅ shader sa stáva OBSERVEROM kamery                 │
│  ✅ shader je PRVOK v observerCollection              │
│  ✅ shader je pretypovaný na Observer*                │
│     (compile-time upcasting)                          │
│  ✅ C++ za behu (runtime) vie, že je to ShaderProgram │
│     a zavolá správnu notify() metódu                  │
│                                                        │
│  JE TO 100% SPRÁVNE! 🎯                               │
└────────────────────────────────────────────────────────┘


COMPILER KONTROLA:
═══════════════════════════════════════════════════

Otázka: Môžem ShaderProgram* pretypovať na Observer*?

Hierarchia dedičnosti:
    Observer  ◄─── Báza
       △
       │ dedí
       │
   ShaderProgram  ◄─── Odvodená

Odpoveď: ✅ ÁNO! ShaderProgram dedí z Observer
         → Implicitné upcasting je BEZPEČNÉ!

ShaderProgram* (0x1000) ──► Observer* (0x1000)
                                       ^^^^^^
                                       Adresa ROVNAKÁ!
                                       Len TYP sa zmenil!



void ShaderProgram::notify(Subject* subject) {
    //                       ^^^^^^^ 
    //                       subject = 0x2000 (adresa Camera)
    //                       Typ: Subject*
    
    // ═══════════════════════════════════════════════════
    // DYNAMIC_CAST #1: Skúsime Camera
    // ═══════════════════════════════════════════════════
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    //               ^^^^^^^^^^^^^^^^^^^^^^^^^^
    //               RUNTIME Type Checking!
}
```

### **6A) Ako funguje dynamic_cast? - DETAILNE**
```
DYNAMIC_CAST ALGORITMUS:
════════════════════════════════════════════════════════

INPUT:
  subject = 0x2000 (Subject*)

KROK 1: C++ pozrie na objekt na adrese 0x2000
────────────────────────────────────────────────────────

Adresa: 0x2000
┌─────────────────────────────────────────────────┐
│ Camera objekt                                   │
├─────────────────────────────────────────────────┤
│ Offset 0: vtable* = 0x5100  ◄─── Načíta vtable │
│ Offset 8: observerCollection...                │
└─────────────────────────────────────────────────┘


KROK 2: C++ načíta RTTI informácie z vtable
────────────────────────────────────────────────────────

Adresa: 0x5100 (VTable pre Camera)
┌─────────────────────────────────────────────────┐
│ VTable pre Camera                               │
├─────────────────────────────────────────────────┤
│ Offset -8: RTTI* ──► 0x6000  ◄─── Type Info!   │
│ [0] ~Camera() ──► 0x9000                        │
│ [1] moveForward() ──► 0x9100                    │
└─────────────────────────────────────────────────┘
         │
         ↓
Adresa: 0x6000 (RTTI - Runtime Type Information)
┌─────────────────────────────────────────────────┐
│ Type Info pre Camera                            │
├─────────────────────────────────────────────────┤
│ name = "Camera"                                 │
│ base_classes = [Subject]                        │
│ ...                                             │
└─────────────────────────────────────────────────┘


KROK 3: C++ porovná typy
────────────────────────────────────────────────────────

Otázka: Je Camera odvodená z Camera? (alebo je to Camera?)
Odpoveď: ✅ ÁNO! Je to PRESNE Camera!

Hierarchia:
    Subject
       △
       │
    Camera  ◄─── Objekt JE Camera!

Výsledok: dynamic_cast USPEJE!


KROK 4: Return hodnota
────────────────────────────────────────────────────────

camera = 0x2000  (Camera*)
         ^^^^^^
         Rovnaká adresa, len iný typ!
         Subject* ──► Camera* ✅
```

### **6B) Vizualizácia dynamic_cast:**
```
PRED dynamic_cast:
════════════════════════════════════════════════════════

subject ──► 0x2000  (typ: Subject*)
            ┌──────────────────────────┐
            │ Camera objekt            │
            │ vtable* ──► "Som Camera" │
            └──────────────────────────┘


dynamic_cast<Camera*>(subject):
════════════════════════════════════════════════════════

1. Zisti typ na adrese 0x2000  ──► "Camera"
2. Je Camera odvodená z Camera? ──► ✅ ÁNO!
3. Return 0x2000 ako Camera*


PO dynamic_cast:
════════════════════════════════════════════════════════

camera ──► 0x2000  (typ: Camera*)  ✅ ÚSPECH!
           ┌──────────────────────────┐
           │ Camera objekt            │
           │ Teraz môžem volať:       │
           │ - getViewMatrix()        │
           │ - getPosition()          │
           └──────────────────────────┘
```

### **6C) Čo keby to bol Light namiesto Camera?**
```
ALTERNATÍVNY SCENÁR:
════════════════════════════════════════════════════════

Predstav si, že subject = 0x3000 (Light objekt)

void ShaderProgram::notify(Subject* subject) {
    // subject = 0x3000 (Light*)
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    //               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //               Skús pretypovať Light na Camera
}


DYNAMIC_CAST ALGORITMUS:
────────────────────────────────────────────────────────

KROK 1: Pozri na objekt na 0x3000
┌─────────────────────────────────────────────────┐
│ Light objekt                                    │
│ vtable* = 0x5200                                │
└─────────────────────────────────────────────────┘

KROK 2: Načítaj RTTI
Adresa: 0x6100 (RTTI pre Light)
┌─────────────────────────────────────────────────┐
│ Type Info pre Light                             │
│ name = "Light"                                  │
│ base_classes = [Subject]                        │
└─────────────────────────────────────────────────┘

KROK 3: Porovnaj typy
Otázka: Je Light odvodený z Camera?
Odpoveď: ❌ NIE! Light a Camera sú SÚRODENCI!

Hierarchia:
    Subject
       △
    ┌──┴──┐
    │     │
  Light Camera
    ❌     
    Nie sú v priamej línii!

Výsledok: dynamic_cast ZLYHÁ!


KROK 4: Return hodnota
────────────────────────────────────────────────────────

camera = nullptr  ❌ ZLYHANIE!





void ShaderProgram::notify(Subject* subject) {
    // ═══════════════════════════════════════════════════
    // SCENÁR 1: subject = Camera (0x2000)
    // ═══════════════════════════════════════════════════
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    // RTTI check: "Je subject typu Camera?"
    // ✅ ÁNO! → camera = 0x2000
    
    if (camera) {  // ✅ TRUE (0x2000 != nullptr)
        updateCameraUniforms(camera);  // VYKONÁ SA!
        //                   ^^^^^^
        //                   Môžem volať Camera metódy:
        //                   camera->getViewMatrix()
        //                   camera->getPosition()
        return;  // ◄─── KONIEC, preskočíme Light
    }
    
    // Táto časť SA NEVYKONÁ (už bol return)
    Light* light = dynamic_cast<Light*>(subject);
    if (light) {
        updateLightUniforms(light);
        return;
    }
    
    
    // ═══════════════════════════════════════════════════
    // SCENÁR 2: subject = Light (0x3000)
    // ═══════════════════════════════════════════════════
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    // RTTI check: "Je subject typu Camera?"
    // ❌ NIE! (je Light) → camera = nullptr
    
    if (camera) {  // ❌ FALSE (nullptr)
        updateCameraUniforms(camera);  // PRESKOČÍ SA!
    }
    
    Light* light = dynamic_cast<Light*>(subject);
    // RTTI check: "Je subject typu Light?"
    // ✅ ÁNO! → light = 0x3000
    
    if (light) {  // ✅ TRUE (0x3000 != nullptr)
        updateLightUniforms(light);  // VYKONÁ SA!
        //                  ^^^^^
        //                  Môžem volať Light metódy:
        //                  light->getPosition()
        //                  light->getColor()
        return;
    }
}
```

**Učiteľ povedal:**
> "Tady máte prímo tu ukázku toho ukázku na ten subjekt, ktorý to pán kolega sa snaží pretypovať na kameru nejdřív, pak na point light"

---

## 📊 KOMPLETNÁ VIZUALIZÁCIA VŠETKÝCH KROKOV
```
ČASOVÁ OS UDALOSTÍ:
════════════════════════════════════════════════════════

T0: Vytvorenie objektov
────────────────────────────────────────────────────────
STACK:                      HEAP:
shader = 0x1000  ──────►   0x1000: ShaderProgram
camera = 0x2000  ──────►   0x2000: Camera


T1: camera->attach(shader)
────────────────────────────────────────────────────────
                           0x2000: Camera
                           ├─ observerCollection:
                           │  └─ data* = 0x4000
                           │
                           └──► 0x4000: [0x1000]
                                        ^^^^^^
                                        Observer*
                                        (ShaderProgram)


T2: camera->moveForward(0.1f)
────────────────────────────────────────────────────────
Camera::moveForward()
  └──► notifyAll()
         └──► for (obs : observerCollection)
                 │
                 obs = 0x1000 (Observer*)
                 │
                 └──► obs->notify(0x2000)
                        │
                        RUNTIME LOOKUP:
                        ├─ Pozri vtable na 0x1000
                        ├─ Nájdi notify() = 0x8000
                        └─ Zavolaj ShaderProgram::notify()


T3: ShaderProgram::notify(0x2000)
────────────────────────────────────────────────────────
                       subject = 0x2000 (Subject*)
                          │
                          ↓
           Camera* camera = dynamic_cast<Camera*>(subject)
                          │
                    RTTI CHECK:
                    ├─ Pozri vtable na 0x2000
                    ├─ Načítaj RTTI: "Camera"
                    ├─ Je Camera == Camera? ✅
                    └─ Return 0x2000 as Camera*
                          │
                          ↓
                    camera = 0x2000 (Camera*)
                          │
                          ↓
                    if (camera) { ✅
                        updateCameraUniforms(camera)
                    }
```

---

## 🎯 ZHRNUTIE S ADRESAMI

| Čo | Typ | Adresa | Ukazuje na |
|-----|-----|--------|------------|
| `shader` | `ShaderProgram*` | `0x1000` | ShaderProgram objekt |
| `camera` | `Camera*` | `0x2000` | Camera objekt |
| `observerCollection[0]` | `Observer*` | `0x1000` | ShaderProgram objekt (rovnaký ako shader!) |
| `observer` (v notifyAll) | `Observer*` | `0x1000` | ShaderProgram objekt |
| `subject` (v notify) | `Subject*` | `0x2000` | Camera objekt |
| `camera` (po dynamic_cast) | `Camera*` | `0x2000` | Camera objekt (rovnaký ako subject!) |

---

## 💡 KĽÚČOVÉ POCHOPENIA

### **1. "Vlk v ovčom rúchu" - Upcasting**
```
shader (ShaderProgram*) ──upcasting──► Observer*
  │                                       │
  0x1000                                 0x1000
  │                                       │
  └───────────────┬───────────────────────┘
                  │
            Oba ukazujú na
            ROVNAKÝ objekt!
```

### **2. Runtime Polymorfizmus - VTable Lookup**
```
observer->notify(this)
    │
    ├─ observer má typ Observer*
    ├─ ALE ukazuje na ShaderProgram objekt
    │
    └──► C++ za behu:
         1. Pozrie vtable na adrese observer
         2. Nájde ShaderProgram::notify()
         3. Zavolá správnu metódu! ✅
```

### **3. Dynamic_cast - RTTI Check**
```
dynamic_cast<Camera*>(subject)
    │
    ├─ subject má typ Subject*
    ├─ Ukazuje na objekt na 0x2000
    │
    └──► C++ za behu:
         1. Načíta RTTI z vtable
         2. Zistí: "Je to Camera!"
         3. Return Camera* (0x2000) ✅
         
         ALEBO
         
         Zistí: "NIE je to Camera!"
         Return nullptr ❌


UPCASTING (compile-time)          DOWNCASTING (runtime)
════════════════════════            ═══════════════════════
ShaderProgram* ──► Observer*        Subject* ──► Camera*
     │                                   │
     └─ Pri attach()                     └─ Pri dynamic_cast
        Automatické                         Musí sa overiť
        Bezpečné ✅                         RTTI kontrola
         


camera->attach(shader);
//             ^^^^^^
//             ShaderProgram* (0x1000)

void Subject::attach(Observer* observer) {
    //                 ^^^^^^^^
    //                 Observer* (0x1000)
    observerCollection.push_back(observer);
}
```
```
HIERARCHIA DEDIČNOSTI:
════════════════════════════════════════════════════════

                  Observer  ◄───────── HORE (báza)
                     △
                     │ dedí
                     │
              ShaderProgram  ◄──────── DOLE (odvodená)


UPCASTING (ShaderProgram* → Observer*):
════════════════════════════════════════════════════════

ShaderProgram* shader = 0x1000
         │
         │ UPCASTING (automatické, compile-time)
         │ Idem HORE v hierarchii
         │ Zo špecifického → na všeobecné
         ↓
Observer* observer = 0x1000  ✅

┌─────────────────────────────────────────────────┐
│ UPCASTING je VŽDY BEZPEČNÉ!                    │
│                                                 │
│ Prečo?                                          │
│ - ShaderProgram JE Observer (dedí z neho)      │
│ - Všetko čo má Observer, má aj ShaderProgram   │
│ - Kompilátor to vie overiť za compile-time     │
│ - Žiadna runtime kontrola nepotrebná           │
└─────────────────────────────────────────────────┘


void ShaderProgram::notify(Subject* subject) {
    //                       ^^^^^^^ 
    //                       Subject* (0x2000)
    
    Camera* camera = dynamic_cast<Camera*>(subject);
    //               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //               Subject* → Camera*
}
```
```
HIERARCHIA DEDIČNOSTI:
════════════════════════════════════════════════════════

                  Subject  ◄──────── HORE (báza)
                     △
                     │ dedí
          ┌──────────┴──────────┐
          │                     │
       Camera                Light  ◄─── DOLE (odvodené)


DOWNCASTING (Subject* → Camera*):
════════════════════════════════════════════════════════

Subject* subject = 0x2000
         │
         │ DOWNCASTING (explicitné, runtime)
         │ Idem DOLE v hierarchii
         │ Zo všeobecného → na špecifické
         │ MUSÍM OVERIŤ! ⚠️
         ↓
Camera* camera = dynamic_cast<Camera*>(subject)
         │
         │ RTTI kontrola za runtime:
         │ "Je subject naozaj Camera?"
         │
         ├─ ✅ ÁNO → camera = 0x2000
         └─ ❌ NIE → camera = nullptr

┌─────────────────────────────────────────────────┐
│ DOWNCASTING je POTENCIÁLNE NEBEZPEČNÉ!         │
│                                                 │
│ Prečo?                                          │
│ - Subject môže byť Camera ALEBO Light          │
│ - Kompilátor to NEVIE overiť za compile-time   │
│ - Potrebujeme RUNTIME kontrolu (RTTI)          │
│ - dynamic_cast zabezpečí bezpečnosť            │
└─────────────────────────────────────────────────┘
```

**Učiteľ povedal:**
> "Tady máte prímo tu ukázku toho ukázku na ten subjekt, ktorý to pán kolega sa snaží pretypovať na kameru nejdřív, pak na point light"

---

## 🔄 Kompletný cyklus: UPCASTING → DOWNCASTING
```
CELÝ ŽIVOTNÝ CYKLUS UKAZOVATEĽA:
════════════════════════════════════════════════════════

T0: Vytvorenie
──────────────────────────────────────────────────────
ShaderProgram* shader = new ShaderProgram();
               ^^^^^^
               Typ: ShaderProgram*
               Adresa: 0x1000


T1: UPCASTING (attach) - COMPILE-TIME
──────────────────────────────────────────────────────
camera->attach(shader);
               ^^^^^^
               ShaderProgram* (0x1000)
                   │
                   │ IMPLICITNÝ UPCASTING
                   │ Kompilátor: "ShaderProgram dedí z Observer,
                   │              môžem automaticky pretypovať"
                   ↓
void attach(Observer* observer)
            ^^^^^^^^
            Observer* (0x1000)
                   │
                   ↓
observerCollection.push_back(observer)
                             ^^^^^^^^
                             Uloží: Observer* (0x1000)

┌─────────────────────────────────────────────────┐
│ Vector teraz obsahuje:                          │
│ [0] = Observer* (0x1000)                        │
│       ale skutočne ukazuje na ShaderProgram!    │
│                                                 │
│ Stratili sme informáciu o type?                │
│ ❌ NIE! Informácia je vo VTABLE!                │
└─────────────────────────────────────────────────┘


T2: RUNTIME POLYMORFIZMUS (notifyAll)
──────────────────────────────────────────────────────
for (Observer* observer : observerCollection) {
     ^^^^^^^^
     Observer* (0x1000)
     
    observer->notify(this);
            ^^^^^^
            Ktorú notify() zavolať?
            
            RUNTIME LOOKUP:
            ├─ Pozri vtable na 0x1000
            ├─ vtable hovorí: "ShaderProgram!"
            └─ Zavolaj ShaderProgram::notify()
}

┌─────────────────────────────────────────────────┐
│ Polymorfizmus zachránil informáciu o type!     │
│ Vďaka vtable C++ vie, že je to ShaderProgram   │
└─────────────────────────────────────────────────┘


T3: DOWNCASTING (notify) - RUNTIME
──────────────────────────────────────────────────────
void ShaderProgram::notify(Subject* subject) {
                            ^^^^^^^ 
                            Subject* (0x2000)
                            Môže to byť Camera alebo Light!
                                │
                                │ EXPLICITNÝ DOWNCASTING
                                │ C++: "Musím overiť RTTI!"
                                ↓
    Camera* camera = dynamic_cast<Camera*>(subject);
                     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                     RTTI kontrola:
                     1. Pozri vtable na 0x2000
                     2. Načítaj type info: "Camera"
                     3. Je Camera == Camera? ✅
                     4. Return Camera* (0x2000)
                                │
                                ↓
    if (camera) {  // ✅ TRUE
        updateCameraUniforms(camera);
        //                   ^^^^^^
        //                   Camera* (0x2000)
    }
}
```

---

## 📋 Porovnanie: UPCASTING vs DOWNCASTING

| Aspekt | UPCASTING | DOWNCASTING |
|--------|-----------|-------------|
| **Smer** | Špecifický → Všeobecný | Všeobecný → Špecifický |
| **Príklad** | `ShaderProgram*` → `Observer*` | `Subject*` → `Camera*` |
| **Kde** | `attach(shader)` | `dynamic_cast<Camera*>(subject)` |
| **Kedy** | Compile-time | Runtime |
| **Syntax** | Implicitné (automatické) | Explicitné (`dynamic_cast`) |
| **Bezpečnosť** | Vždy bezpečné ✅ | Musí sa overiť ⚠️ |
| **Overenie** | Kompilátor | RTTI (Runtime Type Information) |
| **Zlyhanie** | Nemôže zlyhať | Vráti `nullptr` |

---

## 🎨 Analogie zo skutočného sveta

### **UPCASTING = "Prihláška do skupiny"**
```
Predstav si, že:
- ShaderProgram = Konkrétna osoba (Jano)
- Observer = Skupina "Poslucháči"

camera->attach(shader);
               ^^^^^^
               Jano

attach(Observer* observer)
       ^^^^^^^^
       Poslucháč

Jano sa pridáva do skupiny "Poslucháči"
Jano JE poslucháč (dedí z Observer)
✅ Automaticky OK! Netreba nič overovať.

Skupina teraz obsahuje: [Poslucháč Jano]
```

### **DOWNCASTING = "Rozpoznanie konkrétnej osoby"**
```
Predstav si, že:
- subject = "Niekto" (nevieme kto)
- Camera = "Fotograf"
- Light = "Osvetľovač"

notify(Subject* subject)
       ^^^^^^^ 
       "Niekto"

Camera* camera = dynamic_cast<Camera*>(subject)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                 "Je tento Niekto fotograf?"
                 
                 OVERENIE:
                 - Pozrieme sa na identifikačný preukaz
                 - Áno! Je to fotograf ✅
                   → camera = "Fotograf Ján"
                 
                 ALEBO
                 
                 - Nie! Je to osvetľovač ❌
                   → camera = nullptr

// Subject môže byť Camera, Light, Material...
// Ako vieme, ktorý je to?

void ShaderProgram::notify(Subject* subject) {
    // Bez downcastingu:
    // ❌ Nevieme, či volať:
    //    - updateCameraUniforms()
    //    - updateLightUniforms()
    //    - updateMaterialUniforms()
    
    // S downcastingom:
    Camera* camera = dynamic_cast<Camera*>(subject);
    if (camera) {
        updateCameraUniforms(camera);  // ✅ Viem, že je to Camera!
        return;
    }
    
    Light* light = dynamic_cast<Light*>(subject);
    if (light) {
        updateLightUniforms(light);    // ✅ Viem, že je to Light!
        return;
    }
}
```

**Učiteľ povedal:**
> "Tady máte prímo tu ukázku toho ukázku na ten subjekt, ktorý to pán kolega sa snaží pretypovať"

---

## 💡 Je to skutočne opak?

### **ÁNO! Je to INVERZNÁ operácia!**
```
UPCASTING:
════════════════════════════════════════════════════════
Konkrétny typ ──────────► Všeobecný typ
ShaderProgram* ─────────► Observer*
      │                      │
   0x1000                 0x1000
      │                      │
      └──────┬───────────────┘
             │
       Stratíme "špecifickosť"
       Získame "kompatibilitu"


DOWNCASTING:
════════════════════════════════════════════════════════
Všeobecný typ ──────────► Konkrétny typ
Subject* ───────────────► Camera*
    │                        │
 0x2000                   0x2000
    │                        │
    └──────┬─────────────────┘
           │
     Získame "špecifickosť"
     Musíme "overiť"
```

---

## 🎯 Zhrnutie s citátmi učiteľa
```
┌────────────────────────────────────────────────────────┐
│  UPCASTING (compile-time)                              │
├────────────────────────────────────────────────────────┤
│  camera->attach(shader)                                │
│  ShaderProgram* ──► Observer*                          │
│                                                        │
│  ✅ Automatické                                        │
│  ✅ Bezpečné (kompilátor vie overiť)                  │
│  ✅ Idem HORE v hierarchii (špecifický → všeobecný)   │
│                                                        │
│  Učiteľ: "Jenom tam udeláte to, že podjedniť          │
│           z toho subjektu"                             │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│  DOWNCASTING (runtime)                                 │
├────────────────────────────────────────────────────────┤
│  dynamic_cast<Camera*>(subject)                        │
│  Subject* ──► Camera*                                  │
│                                                        │
│  ⚠️  Explicitné (dynamic_cast)                         │
│  ⚠️  Musí sa overiť (RTTI za runtime)                  │
│  ⚠️  Idem DOLE v hierarchii (všeobecný → špecifický)  │
│                                                        │
│  Učiteľ: "ktorý to pán kolega sa snaží pretýpovať     │
│           na kameru nejdřív, pak na point light"       │
└────────────────────────────────────────────────────────┘
                   
                                       

# Dotazy na hodinu ZPG 

Window manager 
Resource Manager - singleton 
Scene Manager 
Camaera Manager 
Light Manager 
