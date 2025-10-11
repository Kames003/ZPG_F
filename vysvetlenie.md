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

prakticky sa jedná o valídne preťaženie nakolko názov metódy ostáva rovnaký mení sa len počat arguemntov a ich datatype 
