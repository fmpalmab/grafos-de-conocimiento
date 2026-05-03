# Laboratorio 4 - Fernando Palma
> Curso MDS7203 · Grafos de Conocimiento

Le pedi a claude que lo dejara bonito.

---

## P1 · Buscar la Universidad de Chile por etiqueta

```sparql
SELECT ?item ?itemLabel WHERE {
  ?item rdfs:label "Universidad de Chile"@es.
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
```

### Resultados

| item | itemLabel |
|------|-----------|
| wd:Q232141 | Universidad de Chile |
| wd:Q3569732 | Universidad de Chile |
| wd:Q4005840 | Universidad de Chile |

> Hay tres entidades con esa etiqueta; la canónica (institución principal) es `wd:Q232141`.

---

## P2 · Clases de la Universidad de Chile (`P31/P279*`)

```sparql
SELECT ?clase ?claseLabel WHERE {
  wd:Q232141 wdt:P31/wdt:P279* ?clase.
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
```

### Resultados (selección)

| clase | claseLabel |
|-------|-----------|
| wd:Q875538 | universidad pública |
| wd:Q3918 | universidad |
| wd:Q23002037 | institución educativa pública |
| wd:Q38723 | institución educativa universitaria |
| wd:Q4671277 | institución académica |
| wd:Q294163 | institución pública |
| wd:Q2385804 | institución educativa |
| wd:Q43229 | organización |
| wd:Q155076 | persona jurídica |
| wd:Q31855 | instituto de investigación |
| wd:Q1664720 | instituto |
| wd:Q2085381 | editorial |
| wd:Q45400320 | editor de acceso abierto |
| wd:Q783794 | compañía |
| wd:Q35120 | entidad |
| wd:Q8425 | sociedad |
| ... | *(jerarquía completa de ~60 clases)* |

> La cadena `P31/P279*` sube toda la jerarquía de subclases, lo que explica la gran cantidad de resultados — desde `universidad pública` hasta abstracciones como `entidad` u `objeto`.

---

## P3 · Universidades sudamericanas fundadas antes que la U. de Chile

```sparql
SELECT ?uni ?uniLabel WHERE {
  ?uni wdt:P31/wdt:P279* wd:Q3918;
       wdt:P17 ?pais;
       wdt:P571 ?fechaUni.

  ?pais wdt:P30 wd:Q18.          -- el país debe estar en América del Sur

  wd:Q232141 wdt:P571 ?fechaUChile.

  FILTER(?fechaUni < ?fechaUChile)

  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
```

> **Nota de implementación:** Las universidades en Wikidata tienen la propiedad `P17` (país), pero no una propiedad directa de continente. Para filtrar por Sudamérica (`wd:Q18`) fue necesario hacer un cruce: primero obtener el país de la universidad y luego verificar que ese país tenga `P30 = wd:Q18` (ubicado en América del Sur). Sin ese paso intermedio no es posible filtrar por continente directamente sobre la universidad.

### Resultados (selección)

| uni | uniLabel |
|-----|---------|
| wd:Q270145 | Universidad Nacional Mayor de San Marcos |
| wd:Q1570489 | Universidad Nacional de Córdoba |
| wd:Q936476 | Universidad Central de Venezuela |
| wd:Q194223 | Universidad de Buenos Aires |
| wd:Q1258413 | Universidad de Antioquia |
| wd:Q1517478 | Pontificia Universidad Javeriana |
| wd:Q760690 | Universidad de Los Andes |
| wd:Q2134722 | Real Universidad de San Felipe |
| wd:Q930279 | Universidad Mayor, Real y Pontificia San Francisco Xavier de Chuquisaca |
| wd:Q3290454 | Universidad Mayor de San Andrés |
| wd:Q1767912 | Universidad Mayor de San Simón |
| wd:Q1634051 | Universidad Nacional de Trujillo |
| wd:Q916367 | Universidad Nacional de San Antonio Abad del Cusco |
| ... | *(lista completa ~40 resultados, con duplicados por múltiples fechas)* |

---

## P4 · Personas que estudiaron en la U. de Chile y en otra universidad chilena

```sparql
SELECT DISTINCT ?persona ?personaLabel ?otraUni ?otraUniLabel WHERE {
  ?persona wdt:P69 wd:Q232141;
           wdt:P69 ?otraUni.
  ?otraUni wdt:P31/wdt:P279* wd:Q3918;
           wdt:P17 wd:Q298.
  FILTER(?otraUni != wd:Q232141)
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
```

### Resultados (selección)

| persona | personaLabel | otraUni | otraUniLabel |
|---------|-------------|---------|-------------|
| wd:Q137217998 | Cecilia Riquelme | wd:Q1129925 | Pontificia Universidad Católica de Chile |
| wd:Q130634916 | Claudia Carrasco Aguilar | wd:Q1129925 | Pontificia Universidad Católica de Chile |
| wd:Q88333905 | Felipe Villanelo | wd:Q1284408 | Pontificia Universidad Católica de Valparaíso |
| wd:Q86437627 | Daniela Valenzuela | wd:Q3138071 | Universidad de Tarapacá |
| wd:Q86437627 | Daniela Valenzuela | wd:Q3244385 | Universidad Católica del Norte |
| wd:Q90802871 | Andrea Lazo | wd:Q457793 | Universidad Técnica Federico Santa María |
| wd:Q88438665 | Eduardo Palma | wd:Q961982 | Universidad Austral de Chile |
| wd:Q6696802 | Luciano Cruz-Coke | wd:Q2374234 | Universidad Academia de Humanismo Cristiano |
| wd:Q6172737 | Álvaro Bisama | wd:Q634259 | Universidad de Playa Ancha |
| wd:Q136601870 | Pablo Cornejo Aguilera | wd:Q940031 | Universidad de los Andes |
| wd:Q108791492 | Katia Trusich | wd:Q611995 | Universidad Adolfo Ibáñez |
| wd:Q59164805 | Macarena Valdés | wd:Q29731 | Universidad Mayor |
| ... | *(~30 resultados)* | | |

---

## P5 · Ocupaciones más frecuentes de egresados de la U. de Chile

```sparql
SELECT ?ocupacion ?ocupacionLabel (COUNT(?persona) AS ?cantidad) WHERE {
  ?persona wdt:P69 wd:Q232141;
           wdt:P106 ?ocupacion.
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
GROUP BY ?ocupacion ?ocupacionLabel
ORDER BY DESC(?cantidad)
```

### Resultados (top 20)

| ocupación | cantidad |
|-----------|----------|
| investigador | 1011 |
| político | 937 |
| abogado | 707 |
| escritor | 243 |
| profesor universitario | 182 |
| médico | 151 |
| empresario | 147 |
| actor | 142 |
| diplomático | 140 |
| economista | 134 |
| periodista | 134 |
| cirujano | 88 |
| poeta | 73 |
| historiador | 72 |
| pintor | 70 |
| ingeniero | 65 |
| juez | 65 |
| académico | 65 |
| arquitecto | 63 |
| ingeniero civil | 62 |
| profesor | 50 |

---

## P6 · Personas de la U. de Chile con premios y año

```sparql
SELECT ?persona ?personaLabel ?premio ?premioLabel ?anio WHERE {
  ?persona wdt:P69 wd:Q232141.
  ?persona p:P166 ?declaracion.
  ?declaracion ps:P166 ?premio.
  OPTIONAL {
    ?declaracion pq:P585 ?fecha.
    BIND(YEAR(?fecha) AS ?anio)
  }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
ORDER BY ?personaLabel
```

### Resultados (selección)

| persona | personaLabel | premio | premioLabel | año |
|---------|-------------|--------|------------|-----|
| wd:Q848 | Arturo Alessandri | wd:Q34473235 | Gran Cruz del Mérito Naval con distintivo blanco | 1922 |
| wd:Q558604 | Luis Barros Borgoño | wd:Q17365974 | Gran Cruz de la Orden de Isabel la Católica | 1896 |
| wd:Q994198 | José Toribio Medina | wd:Q64152520 | Doctorado honoris causa UNAM | 1923 |
| wd:Q99541857 | Eduardo Bunster Montero | wd:Q1316544 | Beca Guggenheim | 1931 |
| wd:Q5408321 | Eugenio Pereira Salas | wd:Q1316544 | Beca Guggenheim | 1933 |
| wd:Q99693572 | Atilio Macchiavello Varas | wd:Q1316544 | Beca Guggenheim | 1934 |
| wd:Q100138716 | Adalberto Steeger Schaeffer | wd:Q1316544 | Beca Guggenheim | 1936 |
| wd:Q359880 | Mariano Latorre | wd:Q1235836 | Premio Atenea | — |
| wd:Q857 | Juan Luis Sanfuentes | wd:Q20886648 | Gran Cruz Orden de Santiago de la Espada | 1921 |
| wd:Q857 | Juan Luis Sanfuentes | wd:Q20866649 | Gran Cruz de la Orden de Carlos III | 1922 |
| ... | *(lista completa, ordenada alfabéticamente)* | | | |

> Se usa el patrón de declaración cualificada (`p:P166` / `ps:P166` / `pq:P585`) para acceder al año del premio, que es un calificador y no está directamente en el triplejo principal.

---

## P7 · Personas asociadas a la U. de Chile (estudiantes o trabajadores)

```sparql
SELECT DISTINCT ?persona ?personaLabel ?rol ?paisLabel ?fechaNac WHERE {
  {
    ?persona wdt:P69 wd:Q232141.
    BIND("estudiante" AS ?rol)
  } UNION {
    ?persona wdt:P108 wd:Q232141.
    BIND("trabajador" AS ?rol)
  }
  OPTIONAL { ?persona wdt:P569 ?fechaNac. }
  OPTIONAL { ?persona wdt:P27 ?pais. }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
```

### Resultados (selección)

| persona | personaLabel | rol | país | fechaNac |
|---------|-------------|-----|------|----------|
| wd:Q5396796 | Aníbal Pinto Santa Cruz | trabajador | Chile | 1 ene 1919 |
| wd:Q516298 | Jaime Eyzaguirre | trabajador | Chile | 21 dic 1908 |
| wd:Q115731 | Gualterio Looser | trabajador | Suiza | 4 sep 1898 |
| wd:Q605194 | Antonio Skármeta | trabajador | Chile | 7 nov 1940 |
| wd:Q451359 | María Isabel Allende | trabajador | Chile | 18 ene 1945 |
| wd:Q264341 | Ruth Cardoso | trabajador | Brasil | 19 sep 1930 |
| wd:Q3754638 | José Balmes | trabajador | Chile | 20 ene 1927 |
| wd:Q5666945 | Alfonso Letelier | trabajador | Chile | 4 oct 1912 |
| wd:Q3155297 | Isidora Aguirre | trabajador | Chile | 22 mar 1919 |
| wd:Q5184445 | Crescente Errázuriz | trabajador | Chile | 18 nov 1839 |
| ... | *(lista extensa)* | | | |

---

## P8 · Egresados vivos cuya edad supera el promedio del grupo

```sparql
SELECT ?persona ?personaLabel ?edad WHERE {

  -- Subconsulta: promedio de edad de egresados vivos
  {
    SELECT (AVG(YEAR(NOW()) - YEAR(?fn)) AS ?promedioEdad) WHERE {
      ?p wdt:P69 wd:Q232141;
         wdt:P569 ?fn.
      FILTER NOT EXISTS { ?p wdt:P570 ?fm. }
    }
  }

  ?persona wdt:P69 wd:Q232141;
           wdt:P569 ?fechaNac.

  FILTER NOT EXISTS { ?persona wdt:P570 ?fechaMuerte. }

  BIND((YEAR(NOW()) - YEAR(?fechaNac)) AS ?edad)

  FILTER(?edad > ?promedioEdad)

  SERVICE wikibase:label { bd:serviceParam wikibase:language "es,en". }
}
ORDER BY DESC(?edad)
```

### Resultados

| persona | personaLabel | edad |
|---------|-------------|------|
| wd:Q5994181 | Manuel Rengifo Vial | 196 |
| wd:Q33126459 | Jaime Eyzaguirre Philippi | 191 |
| wd:Q51881728 | José Nicolás Hurtado de Mendoza y Jaraquemada | 190 |
| wd:Q59543002 | Manuel José Soffia Otaegui | 181 |
| wd:Q5658409 | Adriano Castillo | 159 |
| wd:Q61038481 | Alberto Díaz Lira | 152 |
| wd:Q62568507 | Carlos Cuevas Fernández | 134 |
| wd:Q115243290 | Desiderio Bravo Ortiz | 127 |
| wd:Q2887129 | Gustavo Vicuña Salas | 125 |
| wd:Q5667787 | Alfredo Labbé Villa | 125 |
| wd:Q5920900 | Isabel Viviani | 125 |
| wd:Q16298910 | Juan Pablo Díaz Burgos | 125 |
| wd:Q6453664 | Jorge Lesser | 125 |
| ... | *(~33 resultados)* | |

> **Nota de calidad de datos:** Los resultados incluyen personas con edades implausibles (190+ años) porque el criterio de "vivo" se reduce a la **ausencia de fecha de muerte** (`P570`) en Wikidata. Muchas personas históricas simplemente no tienen ese campo registrado en la base de conocimiento, por lo que el filtro `FILTER NOT EXISTS { ?persona wdt:P570 ?fechaMuerte }` las trata erróneamente como vivas. Esto es una limitación conocida del grafo: la ausencia de un dato no equivale a la negación del hecho (Closed World Assumption vs. Open World Assumption).
