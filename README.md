# Clasificator de Companii pentru Taxonomia de Asigurări - Veridion Challenge

## Despre proiect

Scopul acestui proiect este de a construi un sistem care clasifică corect companiile în una sau mai multe etichete ("labels") dintr-o taxonomie fixă legată de domeniul asigurărilor.

Datele de intrare conțin:

- Descrierea companiei  
- Tag-uri de business  
- Sector, categorie și nișă  

Scopul final este de a atașa fiecărei companii una sau mai multe etichete relevante din taxonomie.

---

## Pașii urmați

### 1. Înțelegerea problemei

Am analizat cerințele și mi-am dat seama că nu trebuie să clasific doar companiile direct legate de asigurări (ex: brokeri, firme de asigurare), ci în general orice firmă care poate fi descrisă printr-o etichetă din taxonomia oferită. Asta include servicii conexe: construcții, agricultură, consultanță, etc.

### 2. Preprocesarea textului

Am combinat toate câmpurile text într-un singur text per companie. Apoi am curățat textul:

- am trecut totul la lowercase  
- am înlocuit cuvinte similare (ex: "insured" devine "insurance")  
- am eliminat caractere speciale și spații inutile  

### 3. Transformare în vectori (embeddings)

Am folosit un model pre-antrenat (`all-mpnet-base-v2`) care transformă textul în vectori numerici. Am generat vectori atât pentru companii, cât și pentru etichetele din taxonomie.

Apoi am calculat similaritatea dintre fiecare companie și fiecare etichetă folosind cosine similarity.

### 4. Boosting pe bază de reguli (label boosting)

Am creat o mapare manuală de la cuvinte cheie în text la etichete. De exemplu, dacă o companie are într-un text cuvântul "roofing", atunci scorul pentru eticheta "Residential Roofing Services" crește puțin.

Boost-ul este mic (0.015) tocmai pentru a evita etichetări greșite.

### 5. Clasificare finală

Pentru fiecare companie:

- alegem toate etichetele cu scor mai mare decât un prag (`threshold = 0.4`)  
- dacă nu e niciuna peste prag, alegem cea mai bună etichetă (fallback)  
- salvăm și top-2 etichete pentru inspecție ulterioară  

---

## De ce am ales această abordare și alte variante considerate

Am optat pentru embedding-uri semantice deoarece oferă o înțelegere mai profundă a textului, comparativ cu metode simple precum TF-IDF, care doar numără cuvinte.

Am încercat și modelul `all-MiniLM-L6-v2`, dar `mpnet` a dat scoruri mai bune.  
N-am ales metode bazate pe fine-tuning pentru că nu aveam un set mare de etichete validate pentru antrenament.  
Ar fi fost complicat și riscant să introduc un model supravegheat pe date potențial greșite.

Am considerat și clasifiere clasice (Random Forest, Logistic Regression) pe TF-IDF, dar am renunțat pentru că nu ar fi înțeles legătura semantică între etichete și text.

Boosting-ul pe reguli m-a ajutat să prioritizez etichete specifice în cazuri ambigue (ex: cuvinte gen “timber”, “roofing”), dar l-am ținut la o intensitate redusă tocmai pentru a nu forța predicții incorecte.

---

## Opțiuni de scalare pe seturi mari de date

- salvam și reutilizăm embedding-urile pentru etichete (nu se schimbă)
- procesam companiile în batch-uri pentru a reduce consumul de memorie
- putem folosi FAISS sau alt sistem de căutare approximate pentru rapiditate
- putem migra pipeline-ul pe un sistem distribuit (Spark, Dask) dacă datele devin foarte mari
