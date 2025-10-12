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

void Application::initialization()
{
    // ... GLFW/GLEW init ...
    
    window = glfwCreateWindow(800, 600, "ZPG project", NULL, NULL);
    
    // ✅ KROK 1: Ulož THIS pointer do okna
    glfwSetWindowUserPointer(window, this);
    //                              ↑
    //                              Pointer na Application inštanciu
}


Čo sa stalo:

GLFW si zapamätá pointer na našu Application inštanciu
Môžeme ho neskôr vyžiadať späť v callbackoch


Krok 2: Lambda wrapper na konverziu statického callbacku
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
Dôležité:

this pointer je implicitný v member funkciách
Môžeme písať camera namiesto this->camera
Máme plný prístup k všetkým member premenným a metódam


Iné callbacky používajú rovnakú techniku:
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

●  ← Center (x, y, z)
       ╱│╲
      ╱ │ ╲  ← Radius
     ●──┼──●
      ╲ │ ╱
       ╲│╱
        ●
B) AABB (Axis-Aligned Bounding Box) - Kocka zarovnaná s osami

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

 D) Mesh Collision (per-triangle)

Testovanie každého trojuholníka modelu
    ┌───┐
    │╲  │  ← 108 vertexov = 36 trojuholníkov
    │ ╲ │     = 36 testov!
    │  ╲│
    └───┘
   
