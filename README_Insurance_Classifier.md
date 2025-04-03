# Clasificator de Companii pentru Taxonomia de Asigurări - Veridion Challenge

## Despre proiect

Scopul acestui proiect este de a construi un sistem care clasifică corect companiile în una sau mai multe etichete ("labels") dintr-o taxonomie fixă legată de domeniul asigurărilor.

Datele de intrare conțin:

- Descrierea companiei
- Tag-uri de business
- Sector, categorie și nișă

Scopul final este de a atașa fiecărei companii una sau mai multe etichete relevante din taxonomie.

## Pașii urmați

### 1. Înțelegerea problemei

Am analizat cerințele și mi-am dat seama că nu trebuie să clasific doar companiile direct legate de asigurări (ex: brokeri, firme de asigurare), ci în general orice firmă care poate fi descrisă printr-o etichetă din taxonomia oferită. Asta include servicii conexe: construcții, agricultură, consultanță, etc.

### 2. Preprocesarea textului

Am combinat toate câmpurile text într-un singur text per companie. Apoi am curățat textul:

- am trecut totul la lowercase,
- am înlocuit cuvinte similare (ex: "insured" devine "insurance"),
- am eliminat caractere speciale și spații inutile.

### 3. Transformare în vectori (embeddings)

Am folosit un model pre-antrenat (all-mpnet-base-v2) care transformă textul în vectori numerici. Am generat vectori atât pentru companii, cât și pentru etichetele din taxonomie.

Apoi am calculat similaritatea dintre fiecare companie și fiecare etichetă (folosind cosine similarity).

### 4. Boosting pe bază de reguli (label boosting)

Am creat o mapare manuală de la cuvinte cheie în text la etichete. De exemplu, dacă o companie are într-un text cuvântul "roofing", atunci scorul pentru eticheta "Residential Roofing Services" crește puțin.

Am limitat boost-ul la o valoare mică (0.015) pentru a evita falsurile pozitive.

### 5. Clasificare finală

Pentru fiecare companie:

- alegem toate etichetele cu scor mai mare decât un prag (0.4)
- dacă nu e niciuna peste prag, alegem cea mai bună etichetă
- salvăm și top-2 etichete pentru inspecție ulterioară

## De ce am ales această abordare și alte variante considerate

Am optat pentru embedding-uri semantice deoarece oferă o înțelegere mai profundă a textului, comparativ cu metode simple precum TF-IDF, care doar numără cuvinte.

Am încercat și modelul `all-MiniLM-L6-v2`, dar `mpnet` a dat scoruri mai bune. N-am ales metode bazate pe fine-tuning pentru că nu aveam un set mare de etichete validate pentru antrenament. Ar fi fost complicat și riscant să introduc un model supravegheat pe date potențial greșite.

Boosting-ul pe reguli ajută mult la clarificarea scorurilor, mai ales pentru cazuri unde embedding-ul e vag. Totuși, l-am setat la o valoare mică pentru a nu genera falsuri pozitive.

## Optiuni de scalare pe seturi mari de date

- salvam si reutilizam embedding-urile pentru etichete (sunt fixe)
- stocam rezultatele incremental, în loc de totul în memorie
- putem folosi un motor de similaritate aproximativă (ex: FAISS) pentru viteză
