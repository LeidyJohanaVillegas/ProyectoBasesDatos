## 🔹 **2. Subconsultas**

1. Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.

   ```
   SELECT 
     p.id,
     p.name,
     p.detail,
     p.price,
     p.category_id,
     p.image
   FROM products p
   JOIN (
     SELECT category_id, AVG(price) AS avg_price
     FROM products
     GROUP BY category_id
   ) avg_cat ON p.category_id = avg_cat.category_id
   WHERE p.price > avg_cat.avg_price;
   +-----+------------------+------------------------------------------------------+-------+-------------+----------------------+
   | id  | name             | detail                                               | price | category_id | image                |
   +-----+------------------+------------------------------------------------------+-------+-------------+----------------------+
   |   1 | Laptop XYZ       | Laptop para trabajo                                  |  2500 |           1 | laptop.jpg           |
   |   3 | Smartphone Alpha | Teléfono inteligente con cámara de 48MP              |  1200 |           1 | smartphone_alpha.jpg |
   |   4 | Monitor 24"      | Monitor Full HD para oficina                         |   350 |           1 | monitor_24.jpg       |
   |   7 | Tablet Z10       | Tablet con pantalla de 10 pulgadas                   |   600 |           1 | tablet_z10.jpg       |
   |   8 | Impresora Láser  | Impresora monocromo de alta velocidad                |   450 |           1 | impresora_laser.jpg  |
   |  12 | Smartwatch S5    | Reloj inteligente con monitor de frecuencia cardíaca |   250 |           1 | smartwatch_s5.jpg    |
   |  21 | Proyector HD     | Proyector portátil para presentaciones               |   350 |           1 | proyector_hd.jpg     |
   | 112 | Producto 112     | Descripción producto 112                             |  14.5 |           2 | NULL                 |
   | 115 | Producto 115     | Producto 115                                         |  16.2 |           2 | prod115.jpg          |
   | 119 | Producto 119     | Producto 119 detalle                                 | 15.75 |           3 | prod119.jpg          |
   | 121 | Producto 121     | Detalle 121                                          |  14.1 |           2 | prod121.jpg          |
   | 128 | Producto 128     | Detalle 128                                          |  14.4 |           3 | NULL                 |
   | 130 | Producto 130     | Detalle final                                        |  16.1 |           2 | NULL                 |
   +-----+------------------+------------------------------------------------------+-------+-------------+----------------------+
   ```

2. Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.

   ```
   SELECT company_id, COUNT(*) AS num_products
   FROM companyproducts
   GROUP BY company_id
   HAVING num_products > (
     SELECT AVG(cnt) FROM (
       SELECT COUNT(*) AS cnt
       FROM companyproducts
       GROUP BY company_id
     ) sub
   );
   +------------+--------------+
   | company_id | num_products |
   +------------+--------------+
   | 900000001  |            5 |
   | 900654321  |            5 |
   | 900987654  |            5 |
   | C001       |            5 |
   | C002       |            4 |
   +------------+--------------+
   ```

3. Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.

   ```
   SELECT p.id, p.name
   FROM products p
   WHERE p.id IN (
       SELECT df.product_id
       FROM favorites f
       JOIN details_favorites df ON f.id = df.favorite_id
   )
   AND p.id IN (
       SELECT qp.product_id
       FROM quality_products qp
   );
   +----+------------+
   | id | name       |
   +----+------------+
   |  1 | Laptop XYZ |
   |  2 | Mouse Pro  |
   +----+------------+
   ```

4. Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.

   ```
   SELECT p.id, p.name, COALESCE(cnt, 0) AS times_favorited
   FROM products p
   LEFT JOIN (
     SELECT product_id, COUNT(*) AS cnt
     FROM details_favorites
     GROUP BY product_id
   ) sub ON p.id = sub.product_id
   ORDER BY times_favorited DESC
   LIMIT 10; -- o sin límite para ver el top
   +-----+------------------------+-----------------+
   | id  | name                   | times_favorited |
   +-----+------------------------+-----------------+
   |   1 | Laptop XYZ             |              11 |
   |   2 | Mouse Pro              |               5 |
   | 999 | Producto sin calificar |               1 |
   |   3 | Smartphone Alpha       |               0 |
   |   4 | Monitor 24"            |               0 |
   |   5 | Teclado Mecánico       |               0 |
   |   6 | Auriculares Bluetooth  |               0 |
   |   7 | Tablet Z10             |               0 |
   |   8 | Impresora Láser        |               0 |
   |   9 | Cámara Web HD          |               0 |
   +-----+------------------------+-----------------+
   ```

5. Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla `rates` ni en `quality_products`.

   ```
   SELECT 
     c.id,
     c.name
   FROM customers c
   WHERE c.email NOT IN (
       SELECT DISTINCT customer_id FROM rates
     )
     AND c.email NOT IN (
       SELECT DISTINCT customer_id FROM quality_products
     );
   +----+-----------------+
   | id | name            |
   +----+-----------------+
   |  1 | Carlos Pérez    |
   |  2 | Ana Gómez       |
   |  3 | Luis Torres     |
   |  4 | María Rivas     |
   |  5 | Diego Mendoza   |
   |  6 | Sofía Castillo  |
   |  7 | Andrés Molina   |
   |  8 | Luisa Fernández |
   |  9 | Pedro López     |
   | 10 | Carolina Reyes  |
   +----+-----------------+
   ```   

6. Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.

   ```
   SELECT p.name
   FROM products p
   JOIN quality_products qp ON p.id = qp.product_id
   GROUP BY p.id, p.name, p.category_id
   HAVING MIN(qp.rating) < (
       SELECT MIN(qp2.rating)
       FROM quality_products qp2
       JOIN products p2 ON qp2.product_id = p2.id
       WHERE p2.category_id = p.category_id
       AND qp2.product_id != p.id
   );
   +---------------+
   | name          |
   +---------------+
   | Cámara Web HD |
   +---------------+
   ```

7. Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.

   ```
   SELECT c.*
   FROM citiesormunicipalities c
   LEFT JOIN customers cu ON cu.city_id = c.code
   WHERE cu.id IS NULL;
   +-------+-----------------------------+-------------+
   | code  | name                        | statereg_id |
   +-------+-----------------------------+-------------+
   | 08372 | Juan de Acosta              | ATL         |
   | 08421 | Luruaco                     | ATL         |
   | 08433 | Malambo                     | ATL         |
   | 08436 | Manatí                      | ATL         |
   | 08520 | Palmar de Varela            | ATL         |
   | 08549 | Piojó                       | ATL         |
   | 08558 | Polonuevo                   | ATL         |
   | 08560 | Ponedera                    | ATL         |
   | 08573 | Puerto Colombia             | ATL         |
   | 08606 | Repelón                     | ATL         |
   | 08634 | Sabanagrande                | ATL         |
   | 08638 | Sabanalarga                 | ATL         |
   | 08675 | Santa Lucía                 | ATL         |
   | 08685 | Santo Tomás                 | ATL         |
   | 08758 | Soledad                     | ATL         |
   | 08770 | Suán                        | ATL         |
   | 08832 | Tubará                      | ATL         |
   | 08849 | Usiacurí                    | ATL         |
   | 13001 | Cartagena                   | BOL         |
   | 13006 | Achí                        | BOL         |
   | 13030 | Altos del Rosario           | BOL         |
   | 13042 | Arenal                      | BOL         |
   | 13052 | Arjona                      | BOL         |
   | 13062 | Arroyohondo                 | BOL         |
   | 13074 | Barranco de Loba            | BOL         |
   | 13140 | Calamar                     | BOL         |
   | 13160 | Cantagallo                  | BOL         |
   | 13188 | Cicuco                      | BOL         |
   | 13212 | Córdoba                     | BOL         |
   | 13222 | Clemencia                   | BOL         |
   | 13244 | El Carmen de Bolívar        | BOL         |
   | 13248 | El Guamo                    | BOL         |
   | 13268 | El Peñón                    | BOL         |
   | 13300 | Hatillo de Loba             | BOL         |
   | 13430 | Magangué                    | BOL         |
   | 13433 | Mahates                     | BOL         |
   | 13440 | Margarita                   | BOL         |
   | 13442 | María la Baja               | BOL         |
   | 13458 | Montecristo                 | BOL         |
   | 13468 | Mompós                      | BOL         |
   | 13473 | Morales                     | BOL         |
   | 13549 | Pinillos                    | BOL         |
   | 13580 | Regidor                     | BOL         |
   | 13600 | Río Viejo                   | BOL         |
   | 13620 | San Cristóbal               | BOL         |
   | 13647 | San Estanislao              | BOL         |
   | 13650 | San Fernando                | BOL         |
   | 13654 | San Jacinto                 | BOL         |
   | 13655 | San Jacinto del Cauca       | BOL         |
   | 13657 | San Juan Nepomuceno         | BOL         |
   | 13667 | San Martín de Loba          | BOL         |
   | 13670 | San Pablo                   | BOL         |
   | 13673 | Santa Catalina              | BOL         |
   | 13683 | Santa Rosa                  | BOL         |
   | 13688 | Santa Rosa del Sur          | BOL         |
   | 13744 | Simití                      | BOL         |
   | 13760 | Soplaviento                 | BOL         |
   | 13780 | Talaigua Nuevo              | BOL         |
   | 13810 | Tiquisio                    | BOL         |
   | 13836 | Turbaco                     | BOL         |
   | 13838 | Turbaná                     | BOL         |
   | 13873 | Villanueva                  | BOL         |
   | 13894 | Zambrano                    | BOL         |
   | 15001 | Tunja                       | BOY         |
   | 15022 | Almeida                     | BOY         |
   | 15047 | Aquitania                   | BOY         |
   | 15051 | Arcabuco                    | BOY         |
   | 15087 | Belén                       | BOY         |
   | 15090 | Berbeo                      | BOY         |
   | 15092 | Betéitiva                   | BOY         |
   | 15097 | Boavita                     | BOY         |
   | 15104 | Boyacá                      | BOY         |
   | 15106 | Briceño                     | BOY         |
   | 15109 | Buenavista                  | BOY         |
   | 15114 | Busbanzá                    | BOY         |
   | 15131 | Caldas                      | BOY         |
   | 15135 | Campohermoso                | BOY         |
   | 15162 | Cerinza                     | BOY         |
   | 15172 | Chinavita                   | BOY         |
   | 15176 | Chiquinquirá                | BOY         |
   | 15180 | Chiscas                     | BOY         |
   | 15183 | Chita                       | BOY         |
   | 15185 | Chitaraque                  | BOY         |
   | 15187 | Chivatá                     | BOY         |
   | 15189 | Ciénega                     | BOY         |
   | 15204 | Cómbita                     | BOY         |
   | 15212 | Coper                       | BOY         |
   | 15215 | Corrales                    | BOY         |
   | 15218 | Covarachía                  | BOY         |
   | 15223 | Cubará                      | BOY         |
   | 15224 | Cucaita                     | BOY         |
   | 15226 | Cuítiva                     | BOY         |
   | 15232 | Chíquiza                    | BOY         |
   | 15236 | Chivor                      | BOY         |
   | 15238 | Duitama                     | BOY         |
   | 15244 | El Cocuy                    | BOY         |
   | 15248 | El Espino                   | BOY         |
   | 15272 | Firavitoba                  | BOY         |
   | 15276 | Floresta                    | BOY         |
   | 15293 | Gachantivá                  | BOY         |
   | 15296 | Gámeza                      | BOY         |
   | 15299 | Garagoa                     | BOY         |
   | 15317 | Guacamayas                  | BOY         |
   | 15322 | Guateque                    | BOY         |
   | 15325 | Guayatá                     | BOY         |
   | 15332 | Güicán                      | BOY         |
   | 15362 | Iza                         | BOY         |
   | 15367 | Jenesano                    | BOY         |
   | 15368 | Jericó                      | BOY         |
   | 15377 | Labranzagrande              | BOY         |
   | 15380 | La Capilla                  | BOY         |
   | 15401 | La Victoria                 | BOY         |
   | 15403 | La Uvita                    | BOY         |
   | 15407 | Villa de Leyva              | BOY         |
   | 15425 | Macanal                     | BOY         |
   | 15442 | Maripí                      | BOY         |
   | 15455 | Miraflores                  | BOY         |
   | 15464 | Mongua                      | BOY         |
   | 15466 | Monguí                      | BOY         |
   | 15469 | Moniquirá                   | BOY         |
   | 15476 | Motavita                    | BOY         |
   | 15480 | Muzo                        | BOY         |
   | 15491 | Nobsa                       | BOY         |
   | 15494 | Nuevo Colón                 | BOY         |
   | 15500 | Oicatá                      | BOY         |
   | 15507 | Otanche                     | BOY         |
   | 15511 | Pachavita                   | BOY         |
   | 15514 | Páez                        | BOY         |
   | 15516 | Paipa                       | BOY         |
   | 15518 | Pajarito                    | BOY         |
   | 15522 | Panqueba                    | BOY         |
   | 15531 | Pauna                       | BOY         |
   | 15533 | Paya                        | BOY         |
   | 15537 | Paz de Río                  | BOY         |
   | 15542 | Pesca                       | BOY         |
   | 15550 | Pisba                       | BOY         |
   | 15572 | Puerto Boyacá               | BOY         |
   | 15580 | Quípama                     | BOY         |
   | 15599 | Ramiriquí                   | BOY         |
   | 15600 | Ráquira                     | BOY         |
   | 15621 | Rondón                      | BOY         |
   | 15632 | Saboyá                      | BOY         |
   | 15638 | Sáchica                     | BOY         |
   | 15646 | Samacá                      | BOY         |
   | 15660 | San Eduardo                 | BOY         |
   | 15664 | San José de Pare            | BOY         |
   | 15667 | San Luis de Gaceno          | BOY         |
   | 15673 | San Mateo                   | BOY         |
   | 15676 | San Miguel de Sema          | BOY         |
   | 15681 | San Pablo de Borbur         | BOY         |
   | 15686 | Santana                     | BOY         |
   | 15690 | Santa María                 | BOY         |
   | 15693 | Santa Rosa de Viterbo       | BOY         |
   | 15696 | Santa Sofía                 | BOY         |
   | 15720 | Sativanorte                 | BOY         |
   | 15723 | Sativasur                   | BOY         |
   | 15740 | Siachoque                   | BOY         |
   | 15753 | Soatá                       | BOY         |
   | 15755 | Socotá                      | BOY         |
   | 15757 | Socha                       | BOY         |
   | 15759 | Sogamoso                    | BOY         |
   | 15761 | Somondoco                   | BOY         |
   | 15762 | Sora                        | BOY         |
   | 15763 | Sotaquirá                   | BOY         |
   | 15764 | Soracá                      | BOY         |
   | 15774 | Susacón                     | BOY         |
   | 15776 | Sutamarchán                 | BOY         |
   | 15778 | Sutatenza                   | BOY         |
   | 15790 | Tasco                       | BOY         |
   | 15798 | Tenza                       | BOY         |
   | 15804 | Tibaná                      | BOY         |
   | 15806 | Tibasosa                    | BOY         |
   | 15808 | Tinjacá                     | BOY         |
   | 15810 | Tipacoque                   | BOY         |
   | 15814 | Toca                        | BOY         |
   | 15816 | Togüí                       | BOY         |
   | 15820 | Tópaga                      | BOY         |
   | 15822 | Tota                        | BOY         |
   | 15832 | Tunungua                    | BOY         |
   | 15835 | Turmeque                    | BOY         |
   | 15837 | Tuta                        | BOY         |
   | 15839 | Tutazá                      | BOY         |
   | 15842 | Úmbita                      | BOY         |
   | 15861 | Ventaquemada                | BOY         |
   | 15879 | Viracachá                   | BOY         |
   | 15897 | Zetaquira                   | BOY         |
   | 17001 | Manizales                   | CAL         |
   | 17013 | Aguadas                     | CAL         |
   | 17042 | Anserma                     | CAL         |
   | 17050 | Aranzazu                    | CAL         |
   | 17088 | Belalcázar                  | CAL         |
   | 17174 | Chinchiná                   | CAL         |
   | 17272 | Filadelfia                  | CAL         |
   | 17380 | La Dorada                   | CAL         |
   | 17388 | La Merced                   | CAL         |
   | 17433 | Manzanares                  | CAL         |
   | 17442 | Marmato                     | CAL         |
   | 17444 | Marquetalia                 | CAL         |
   | 17446 | Marulanda                   | CAL         |
   | 17486 | Neira                       | CAL         |
   | 17495 | Norcasia                    | CAL         |
   | 17513 | Pácora                      | CAL         |
   | 17524 | Palestina                   | CAL         |
   | 17541 | Pensilvania                 | CAL         |
   | 17614 | Riosucio                    | CAL         |
   | 17616 | Risaralda                   | CAL         |
   | 17653 | Salamina                    | CAL         |
   | 17662 | Samaná                      | CAL         |
   | 17665 | San José                    | CAL         |
   | 17777 | Supía                       | CAL         |
   | 17867 | Victoria                    | CAL         |
   | 17873 | Villamaría                  | CAL         |
   | 17877 | Viterbo                     | CAL         |
   | 18001 | Florencia                   | CAQ         |
   | 18029 | Albania                     | CAQ         |
   | 18094 | Belén de los Andaquíes      | CAQ         |
   | 18150 | Cartagena del Chairá        | CAQ         |
   | 18205 | Curillo                     | CAQ         |
   | 18247 | El Doncello                 | CAQ         |
   | 18256 | El Paujil                   | CAQ         |
   | 18410 | La Montañita                | CAQ         |
   | 18460 | Milán                       | CAQ         |
   | 18479 | Morelia                     | CAQ         |
   | 18592 | Puerto Rico                 | CAQ         |
   | 18610 | San José de la Fragua       | CAQ         |
   | 18753 | San Vicente del Caguán      | CAQ         |
   | 18756 | Solano                      | CAQ         |
   | 18785 | Solita                      | CAQ         |
   | 18860 | Valparaíso                  | CAQ         |
   | 19001 | Popayán                     | CAU         |
   | 19022 | Almaguer                    | CAU         |
   | 19050 | Argelia                     | CAU         |
   | 19075 | Balboa                      | CAU         |
   | 19100 | Bolívar                     | CAU         |
   | 19110 | Buenos Aires                | CAU         |
   | 19130 | Cajibío                     | CAU         |
   | 19137 | Caldono                     | CAU         |
   | 19142 | Caloto                      | CAU         |
   | 19212 | Corinto                     | CAU         |
   | 19256 | El Tambo                    | CAU         |
   | 19290 | Florencia                   | CAU         |
   | 19318 | Guapi                       | CAU         |
   | 19355 | Inzá                        | CAU         |
   | 19364 | Jambaló                     | CAU         |
   | 19392 | La Sierra                   | CAU         |
   | 19397 | La Vega                     | CAU         |
   | 19418 | López                       | CAU         |
   | 19450 | Mercaderes                  | CAU         |
   | 19455 | Miranda                     | CAU         |
   | 19473 | Morales                     | CAU         |
   | 19513 | Padilla                     | CAU         |
   | 19517 | Páez                        | CAU         |
   | 19532 | Patía                       | CAU         |
   | 19533 | Piamonte                    | CAU         |
   | 19548 | Piendamó                    | CAU         |
   | 19573 | Puerto Tejada               | CAU         |
   | 19585 | Puracé                      | CAU         |
   | 19622 | Rosas                       | CAU         |
   | 19693 | San Sebastián               | CAU         |
   | 19698 | Santander de Quilichao      | CAU         |
   | 19701 | Santa Rosa                  | CAU         |
   | 19743 | Silvia                      | CAU         |
   | 19760 | Sotará                      | CAU         |
   | 19780 | Suárez                      | CAU         |
   | 19785 | Sucre                       | CAU         |
   | 19807 | Timbío                      | CAU         |
   | 19809 | Timbiquí                    | CAU         |
   | 19821 | Toribío                     | CAU         |
   | 19824 | Totoró                      | CAU         |
   | 19845 | Villa Rica                  | CAU         |
   | 20001 | Valledupar                  | CES         |
   | 20011 | Aguachica                   | CES         |
   | 20013 | Agustín Codazzi             | CES         |
   | 20032 | Astrea                      | CES         |
   | 20045 | Becerril                    | CES         |
   | 20060 | Bosconia                    | CES         |
   | 20175 | Chimichagua                 | CES         |
   | 20178 | Chiriguaná                  | CES         |
   | 20228 | Curumaní                    | CES         |
   | 20238 | El Copey                    | CES         |
   | 20250 | El Paso                     | CES         |
   | 20295 | Gamarra                     | CES         |
   | 20310 | González                    | CES         |
   | 20383 | La Gloria                   | CES         |
   | 20400 | La Jagua de Ibirico         | CES         |
   | 20443 | Manaure                     | CES         |
   | 20517 | Pailitas                    | CES         |
   | 20550 | Pelaya                      | CES         |
   | 20570 | Pueblo Bello                | CES         |
   | 20614 | Río de Oro                  | CES         |
   | 20621 | La Paz                      | CES         |
   | 20710 | San Alberto                 | CES         |
   | 20750 | San Diego                   | CES         |
   | 20770 | San Martín                  | CES         |
   | 20787 | Tamalameque                 | CES         |
   | 23001 | Montería                    | COR         |
   | 23068 | Ayapel                      | COR         |
   | 23079 | Buenavista                  | COR         |
   | 23090 | Canalete                    | COR         |
   | 23162 | Cereté                      | COR         |
   | 23168 | Chimá                       | COR         |
   | 23182 | Chinú                       | COR         |
   | 23189 | Ciénaga de Oro              | COR         |
   | 23300 | Cotorra                     | COR         |
   | 23350 | La Apartada                 | COR         |
   | 23417 | Lorica                      | COR         |
   | 23419 | Los Córdobas                | COR         |
   | 23464 | Momil                       | COR         |
   | 23466 | Montelíbano                 | COR         |
   | 23500 | Moñitos                     | COR         |
   | 23555 | Planeta Rica                | COR         |
   | 23570 | Pueblo Nuevo                | COR         |
   | 23574 | Puerto Escondido            | COR         |
   | 23580 | Puerto Libertador           | COR         |
   | 23586 | Purísima                    | COR         |
   | 23660 | Sahagún                     | COR         |
   | 23670 | San Andrés de Sotavento     | COR         |
   | 23672 | San Antero                  | COR         |
   | 23675 | San Bernardo del Viento     | COR         |
   | 23678 | San Carlos                  | COR         |
   | 23686 | San Pelayo                  | COR         |
   | 23807 | Tierralta                   | COR         |
   | 23855 | Valencia                    | COR         |
   | 25001 | Agua de Dios                | CUN         |
   | 25019 | Albán                       | CUN         |
   | 25035 | Anapoima                    | CUN         |
   | 25040 | Anolaima                    | CUN         |
   | 25053 | Arbeláez                    | CUN         |
   | 25086 | Beltrán                     | CUN         |
   | 25095 | Bituima                     | CUN         |
   | 25099 | Bojacá                      | CUN         |
   | 25120 | Cabrera                     | CUN         |
   | 25123 | Cachipay                    | CUN         |
   | 25126 | Cajicá                      | CUN         |
   | 25148 | Caparrapí                   | CUN         |
   | 25151 | Cáqueza                     | CUN         |
   | 25154 | Carmen de Carupa            | CUN         |
   | 25168 | Chaguaní                    | CUN         |
   | 25175 | Chía                        | CUN         |
   | 25178 | Chipaque                    | CUN         |
   | 25181 | Choachí                     | CUN         |
   | 25183 | Chocontá                    | CUN         |
   | 25200 | Cogua                       | CUN         |
   | 25214 | Cota                        | CUN         |
   | 25224 | Cucunubá                    | CUN         |
   | 25245 | El Colegio                  | CUN         |
   | 25258 | El Peñón                    | CUN         |
   | 25260 | El Rosal                    | CUN         |
   | 25269 | Facatativá                  | CUN         |
   | 25279 | Fómeque                     | CUN         |
   | 25281 | Fosca                       | CUN         |
   | 25286 | Funza                       | CUN         |
   | 25288 | Fúquene                     | CUN         |
   | 25290 | Fusagasugá                  | CUN         |
   | 25293 | Gachalá                     | CUN         |
   | 25295 | Gachancipá                  | CUN         |
   | 25297 | Gachetá                     | CUN         |
   | 25299 | Gama                        | CUN         |
   | 25307 | Girardot                    | CUN         |
   | 25312 | Granada                     | CUN         |
   | 25317 | Guachetá                    | CUN         |
   | 25320 | Guaduas                     | CUN         |
   | 25322 | Guasca                      | CUN         |
   | 25324 | Guataquí                    | CUN         |
   | 25326 | Guatavita                   | CUN         |
   | 25328 | Guayabal de Síquima         | CUN         |
   | 25335 | Guayabetal                  | CUN         |
   | 25339 | Gutiérrez                   | CUN         |
   | 25368 | Jerusalén                   | CUN         |
   | 25372 | Junín                       | CUN         |
   | 25377 | La Calera                   | CUN         |
   | 25386 | La Mesa                     | CUN         |
   | 25394 | La Palma                    | CUN         |
   | 25398 | La Peña                     | CUN         |
   | 25402 | La Vega                     | CUN         |
   | 25407 | Lenguazaque                 | CUN         |
   | 25426 | Macheta                     | CUN         |
   | 25430 | Madrid                      | CUN         |
   | 25436 | Manta                       | CUN         |
   | 25438 | Medina                      | CUN         |
   | 25473 | Mosquera                    | CUN         |
   | 25483 | Nariño                      | CUN         |
   | 25486 | Nemocón                     | CUN         |
   | 25488 | Nilo                        | CUN         |
   | 25489 | Nimaima                     | CUN         |
   | 25491 | Nocaima                     | CUN         |
   | 25506 | Venecia                     | CUN         |
   | 25513 | Pacho                       | CUN         |
   | 25518 | Paime                       | CUN         |
   | 25524 | Pandi                       | CUN         |
   | 25530 | Paratebueno                 | CUN         |
   | 25535 | Pasca                       | CUN         |
   | 25572 | Puerto Salgar               | CUN         |
   | 25580 | Pulí                        | CUN         |
   | 25592 | Quebradanegra               | CUN         |
   | 25594 | Quetame                     | CUN         |
   | 25596 | Quipile                     | CUN         |
   | 25599 | Apulo                       | CUN         |
   | 25612 | Ricaurte                    | CUN         |
   | 25645 | San Antonio del Tequendama  | CUN         |
   | 25649 | San Bernardo                | CUN         |
   | 25653 | San Cayetano                | CUN         |
   | 25658 | San Francisco               | CUN         |
   | 25662 | San Juan de Río Seco        | CUN         |
   | 25718 | Sasaima                     | CUN         |
   | 25736 | Sesquilé                    | CUN         |
   | 25740 | Sibaté                      | CUN         |
   | 25743 | Silvania                    | CUN         |
   | 25745 | Simijaca                    | CUN         |
   | 25754 | Soacha                      | CUN         |
   | 25758 | Sopó                        | CUN         |
   | 25769 | Subachoque                  | CUN         |
   | 25772 | Suesca                      | CUN         |
   | 25777 | Supatá                      | CUN         |
   | 25779 | Susa                        | CUN         |
   | 25781 | Sutatausa                   | CUN         |
   | 25785 | Tabio                       | CUN         |
   | 25793 | Tausa                       | CUN         |
   | 25797 | Tena                        | CUN         |
   | 25799 | Tenjo                       | CUN         |
   | 25805 | Tibacuy                     | CUN         |
   | 25807 | Tibirita                    | CUN         |
   | 25815 | Tocaima                     | CUN         |
   | 25817 | Tocancipá                   | CUN         |
   | 25823 | Topaipí                     | CUN         |
   | 25839 | Ubalá                       | CUN         |
   | 25841 | Ubaque                      | CUN         |
   | 25843 | Villa de San Diego de Ubaté | CUN         |
   | 25845 | Une                         | CUN         |
   | 25851 | Útica                       | CUN         |
   | 25862 | Vergara                     | CUN         |
   | 25867 | Vianí                       | CUN         |
   | 25871 | Villagómez                  | CUN         |
   | 25873 | Villapinzón                 | CUN         |
   | 25875 | Villeta                     | CUN         |
   | 25878 | Viotá                       | CUN         |
   | 25885 | Yacopí                      | CUN         |
   | 25898 | Zipacón                     | CUN         |
   | 25899 | Zipaquirá                   | CUN         |
   | 27001 | Quibdó                      | CHO         |
   | 27006 | Acandí                      | CHO         |
   | 27025 | Alto Baudó                  | CHO         |
   | 27050 | Atrato                      | CHO         |
   | 27073 | Bagadó                      | CHO         |
   | 27075 | Bahía Solano                | CHO         |
   | 27077 | Bajo Baudó                  | CHO         |
   | 27086 | Belén de Bajirá             | CHO         |
   | 27099 | Bojayá                      | CHO         |
   | 27135 | El Cantón del San Pablo     | CHO         |
   | 27150 | Carmen del Darién           | CHO         |
   | 27160 | Cértegui                    | CHO         |
   | 27205 | Condoto                     | CHO         |
   | 27245 | El Carmen de Atrato         | CHO         |
   | 27250 | El Litoral del San Juan     | CHO         |
   | 27361 | Istmina                     | CHO         |
   | 27372 | Juradó                      | CHO         |
   | 27413 | Lloró                       | CHO         |
   | 27425 | Medio Atrato                | CHO         |
   | 27430 | Medio Baudó                 | CHO         |
   | 27450 | Medio San Juan              | CHO         |
   | 27491 | Nóvita                      | CHO         |
   | 27495 | Nuquí                       | CHO         |
   | 27580 | Río Iró                     | CHO         |
   | 27600 | Río Quito                   | CHO         |
   | 27615 | Riosucio                    | CHO         |
   | 27660 | San José del Palmar         | CHO         |
   | 27745 | Sipí                        | CHO         |
   | 27787 | Tadó                        | CHO         |
   | 27800 | Unguía                      | CHO         |
   | 27810 | Unión Panamericana          | CHO         |
   | 41001 | Neiva                       | HUI         |
   | 41006 | Acevedo                     | HUI         |
   | 41013 | Agrado                      | HUI         |
   | 41016 | Aipe                        | HUI         |
   | 41020 | Algeciras                   | HUI         |
   | 41026 | Altamira                    | HUI         |
   | 41078 | Baraya                      | HUI         |
   | 41132 | Campoalegre                 | HUI         |
   | 41206 | Colombia                    | HUI         |
   | 41244 | Elías                       | HUI         |
   | 41298 | Garzón                      | HUI         |
   | 41306 | Gigante                     | HUI         |
   | 41319 | Guadalupe                   | HUI         |
   | 41349 | Hobo                        | HUI         |
   | 41357 | Íquira                      | HUI         |
   | 41359 | Isnos                       | HUI         |
   | 41378 | La Argentina                | HUI         |
   | 41396 | La Plata                    | HUI         |
   | 41483 | Nátaga                      | HUI         |
   | 41503 | Oporapa                     | HUI         |
   | 41518 | Paicol                      | HUI         |
   | 41524 | Palermo                     | HUI         |
   | 41530 | Palestina                   | HUI         |
   | 41548 | Pital                       | HUI         |
   | 41551 | Pitalito                    | HUI         |
   | 41615 | Rivera                      | HUI         |
   | 41660 | Saladoblanco                | HUI         |
   | 41668 | San Agustín                 | HUI         |
   | 41676 | Santa María                 | HUI         |
   | 41770 | Suaza                       | HUI         |
   | 41791 | Tarqui                      | HUI         |
   | 41797 | Tesalia                     | HUI         |
   | 41799 | Tello                       | HUI         |
   | 41801 | Teruel                      | HUI         |
   | 41807 | Timaná                      | HUI         |
   | 41872 | Villavieja                  | HUI         |
   | 41885 | Yaguará                     | HUI         |
   | 44001 | Riohacha                    | LAG         |
   | 44035 | Albania                     | LAG         |
   | 44078 | Barrancas                   | LAG         |
   | 44090 | Dibulla                     | LAG         |
   | 44098 | Distracción                 | LAG         |
   | 44110 | El Molino                   | LAG         |
   | 44279 | Fonseca                     | LAG         |
   | 44378 | Hatonuevo                   | LAG         |
   | 44420 | La Jagua del Pilar          | LAG         |
   | 44430 | Maicao                      | LAG         |
   | 44560 | Manaure                     | LAG         |
   | 44650 | San Juan del Cesar          | LAG         |
   | 44847 | Uribia                      | LAG         |
   | 44855 | Urumita                     | LAG         |
   | 44874 | Villanueva                  | LAG         |
   | 47001 | Santa Marta                 | MAG         |
   | 47030 | Algarrobo                   | MAG         |
   | 47053 | Aracataca                   | MAG         |
   | 47058 | Ariguaní                    | MAG         |
   | 47161 | Cerro San Antonio           | MAG         |
   | 47170 | Chivolo                     | MAG         |
   | 47189 | Ciénaga                     | MAG         |
   | 47205 | Concordia                   | MAG         |
   | 47245 | El Banco                    | MAG         |
   | 47258 | El Piñón                    | MAG         |
   | 47268 | El Retén                    | MAG         |
   | 47288 | Fundación                   | MAG         |
   | 47318 | Guamal                      | MAG         |
   | 47460 | Nueva Granada               | MAG         |
   | 47541 | Pedraza                     | MAG         |
   | 47545 | Pijiño del Carmen           | MAG         |
   | 47551 | Pivijay                     | MAG         |
   | 47555 | Plato                       | MAG         |
   | 47570 | Puebloviejo                 | MAG         |
   | 47605 | Remolino                    | MAG         |
   | 47660 | Sabanas de San Ángel        | MAG         |
   | 47675 | Salamina                    | MAG         |
   | 47692 | San Sebastián               | MAG         |
   | 47703 | San Zenón                   | MAG         |
   | 47707 | Santa Ana                   | MAG         |
   | 47720 | Santa Bárbara de Pinto      | MAG         |
   | 47745 | Sitionuevo                  | MAG         |
   | 47798 | Tenerife                    | MAG         |
   | 47960 | Zapayán                     | MAG         |
   | 47980 | Zona Bananera               | MAG         |
   | 50001 | Villavicencio               | MET         |
   | 50006 | Acacías                     | MET         |
   | 50110 | Barranca de Upía            | MET         |
   | 50124 | Cabuyaro                    | MET         |
   | 50150 | Castilla la Nueva           | MET         |
   | 50223 | Cubarral                    | MET         |
   | 50226 | Cumaral                     | MET         |
   | 50245 | El Calvario                 | MET         |
   | 50251 | El Castillo                 | MET         |
   | 50270 | El Dorado                   | MET         |
   | 50287 | Fuente de Oro               | MET         |
   | 50313 | Granada                     | MET         |
   | 50318 | Guamal                      | MET         |
   | 50325 | Mapiripán                   | MET         |
   | 50330 | Mesetas                     | MET         |
   | 50350 | La Macarena                 | MET         |
   | 50370 | Uribe                       | MET         |
   | 50400 | Lejanías                    | MET         |
   | 50450 | Puerto Concordia            | MET         |
   | 50568 | Puerto Gaitán               | MET         |
   | 50573 | Puerto López                | MET         |
   | 50577 | Puerto Lleras               | MET         |
   | 50590 | Puerto Rico                 | MET         |
   | 50606 | Restrepo                    | MET         |
   | 50680 | San Carlos de Guaroa        | MET         |
   | 50683 | San Juan de Arama           | MET         |
   | 50686 | San Juanito                 | MET         |
   | 50689 | San Martín                  | MET         |
   | 50711 | Vista Hermosa               | MET         |
   | 52001 | Pasto                       | NAR         |
   | 52019 | Albán                       | NAR         |
   | 52022 | Aldana                      | NAR         |
   | 52036 | Ancuyá                      | NAR         |
   | 52051 | Arboleda                    | NAR         |
   | 52079 | Barbacoas                   | NAR         |
   | 52083 | Belén                       | NAR         |
   | 52110 | Buesaco                     | NAR         |
   | 52203 | Colón                       | NAR         |
   | 52207 | Consacá                     | NAR         |
   | 52210 | Contadero                   | NAR         |
   | 52215 | Córdoba                     | NAR         |
   | 52224 | Cuaspud                     | NAR         |
   | 52227 | Cumbal                      | NAR         |
   | 52233 | Cumbitara                   | NAR         |
   | 52240 | Chachagüí                   | NAR         |
   | 52250 | El Charco                   | NAR         |
   | 52254 | El Peñol                    | NAR         |
   | 52256 | El Rosario                  | NAR         |
   | 52258 | El Tablón de Gómez          | NAR         |
   | 52260 | El Tambo                    | NAR         |
   | 52287 | Funes                       | NAR         |
   | 52317 | Guachucal                   | NAR         |
   | 52320 | Guaitarilla                 | NAR         |
   | 52323 | Gualmatán                   | NAR         |
   | 52352 | Iles                        | NAR         |
   | 52354 | Imués                       | NAR         |
   | 52356 | Ipiales                     | NAR         |
   | 52378 | La Cruz                     | NAR         |
   | 52381 | La Florida                  | NAR         |
   | 52385 | La Llanada                  | NAR         |
   | 52390 | La Tola                     | NAR         |
   | 52399 | La Unión                    | NAR         |
   | 52405 | Leiva                       | NAR         |
   | 52411 | Linares                     | NAR         |
   | 52418 | Los Andes                   | NAR         |
   | 52427 | Magüí                       | NAR         |
   | 52435 | Mallama                     | NAR         |
   | 52473 | Mosquera                    | NAR         |
   | 52480 | Nariño                      | NAR         |
   | 52490 | Olaya Herrera               | NAR         |
   | 52506 | Ospina                      | NAR         |
   | 52520 | Francisco Pizarro           | NAR         |
   | 52540 | Policarpa                   | NAR         |
   | 52560 | Potosí                      | NAR         |
   | 52565 | Providencia                 | NAR         |
   | 52573 | Puerres                     | NAR         |
   | 52585 | Pupiales                    | NAR         |
   | 52612 | Ricaurte                    | NAR         |
   | 52621 | Roberto Payán               | NAR         |
   | 52678 | Samaniego                   | NAR         |
   | 52683 | Sandoná                     | NAR         |
   | 52685 | San Bernardo                | NAR         |
   | 52687 | San Lorenzo                 | NAR         |
   | 52693 | San Pablo                   | NAR         |
   | 52694 | San Pedro de Cartago        | NAR         |
   | 52696 | Santa Bárbara               | NAR         |
   | 52699 | Santacruz                   | NAR         |
   | 52720 | Sapuyes                     | NAR         |
   | 52786 | Taminango                   | NAR         |
   | 52788 | Tangua                      | NAR         |
   | 52835 | Tumaco                      | NAR         |
   | 52838 | Tuquerres                   | NAR         |
   | 52885 | Yacuanquer                  | NAR         |
   | 54001 | Cúcuta                      | NSA         |
   | 54003 | Ábrego                      | NSA         |
   | 54051 | Arboledas                   | NSA         |
   | 54099 | Bochalema                   | NSA         |
   | 54109 | Bucarasica                  | NSA         |
   | 54125 | Cácota                      | NSA         |
   | 54128 | Cáchira                     | NSA         |
   | 54172 | Chinácota                   | NSA         |
   | 54174 | Chitagá                     | NSA         |
   | 54206 | Convención                  | NSA         |
   | 54223 | Cucutilla                   | NSA         |
   | 54239 | Durania                     | NSA         |
   | 54245 | El Carmen                   | NSA         |
   | 54250 | El Tarra                    | NSA         |
   | 54261 | El Zulia                    | NSA         |
   | 54313 | Gramalote                   | NSA         |
   | 54344 | Hacarí                      | NSA         |
   | 54347 | Herrán                      | NSA         |
   | 54377 | Labateca                    | NSA         |
   | 54385 | La Esperanza                | NSA         |
   | 54398 | La Playa                    | NSA         |
   | 54405 | Los Patios                  | NSA         |
   | 54418 | Lourdes                     | NSA         |
   | 54480 | Mutiscua                    | NSA         |
   | 54498 | Ocaña                       | NSA         |
   | 54518 | Pamplona                    | NSA         |
   | 54520 | Pamplonita                  | NSA         |
   | 54553 | Puerto Santander            | NSA         |
   | 54599 | Ragonvalia                  | NSA         |
   | 54660 | Salazar                     | NSA         |
   | 54670 | San Calixto                 | NSA         |
   | 54673 | San Cayetano                | NSA         |
   | 54680 | Santiago                    | NSA         |
   | 54720 | Sardinata                   | NSA         |
   | 54743 | Silos                       | NSA         |
   | 54800 | Teorama                     | NSA         |
   | 54810 | Tibú                        | NSA         |
   | 54820 | Toledo                      | NSA         |
   | 54871 | Villa Caro                  | NSA         |
   | 54874 | Villa del Rosario           | NSA         |
   | 63001 | Armenia                     | QUI         |
   | 63111 | Buenavista                  | QUI         |
   | 63130 | Calarcá                     | QUI         |
   | 63190 | Circasia                    | QUI         |
   | 63212 | Córdoba                     | QUI         |
   | 63272 | Filandia                    | QUI         |
   | 63302 | Génova                      | QUI         |
   | 63401 | La Tebaida                  | QUI         |
   | 63470 | Montenegro                  | QUI         |
   | 63548 | Pijao                       | QUI         |
   | 63594 | Quimbaya                    | QUI         |
   | 63690 | Salento                     | QUI         |
   | 66001 | Pereira                     | RIS         |
   | 66045 | Apía                        | RIS         |
   | 66075 | Balboa                      | RIS         |
   | 66088 | Belén de Umbría             | RIS         |
   | 66170 | Dosquebradas                | RIS         |
   | 66318 | Guática                     | RIS         |
   | 66383 | La Celia                    | RIS         |
   | 66400 | La Virginia                 | RIS         |
   | 66440 | Marsella                    | RIS         |
   | 66456 | Mistrató                    | RIS         |
   | 66572 | Pueblo Rico                 | RIS         |
   | 66594 | Quinchía                    | RIS         |
   | 66682 | Santa Rosa de Cabal         | RIS         |
   | 66687 | Santuario                   | RIS         |
   | 68001 | Bucaramanga                 | SAN         |
   | 68013 | Aguada                      | SAN         |
   | 68020 | Albania                     | SAN         |
   | 68051 | Aratoca                     | SAN         |
   | 68077 | Barbosa                     | SAN         |
   | 68079 | Barichara                   | SAN         |
   | 68081 | Barrancabermeja             | SAN         |
   | 68092 | Betulia                     | SAN         |
   | 68101 | Bolívar                     | SAN         |
   | 68121 | Cabrera                     | SAN         |
   | 68132 | California                  | SAN         |
   | 68147 | Capitanejo                  | SAN         |
   | 68152 | Carcasí                     | SAN         |
   | 68160 | Cepitá                      | SAN         |
   | 68162 | Cerrito                     | SAN         |
   | 68167 | Charalá                     | SAN         |
   | 68169 | Charta                      | SAN         |
   | 68176 | Chima                       | SAN         |
   | 68179 | Chipatá                     | SAN         |
   | 68190 | Cimitarra                   | SAN         |
   | 68207 | Concepción                  | SAN         |
   | 68209 | Confines                    | SAN         |
   | 68211 | Contratación                | SAN         |
   | 68217 | Coromoro                    | SAN         |
   | 68229 | Curití                      | SAN         |
   | 68235 | El Carmen de Chucurí        | SAN         |
   | 68245 | El Guacamayo                | SAN         |
   | 68250 | El Peñón                    | SAN         |
   | 68255 | El Playón                   | SAN         |
   | 68264 | Encino                      | SAN         |
   | 68266 | Enciso                      | SAN         |
   | 68271 | Florián                     | SAN         |
   | 68276 | Floridablanca               | SAN         |
   | 68296 | Galán                       | SAN         |
   | 68298 | Gambita                     | SAN         |
   | 68307 | Girón                       | SAN         |
   | 68318 | Guaca                       | SAN         |
   | 68320 | Guadalupe                   | SAN         |
   | 68322 | Guapotá                     | SAN         |
   | 68324 | Guavatá                     | SAN         |
   | 68327 | Güepsa                      | SAN         |
   | 68344 | Hato                        | SAN         |
   | 68368 | Jesús María                 | SAN         |
   | 68370 | Jordán                      | SAN         |
   | 68377 | La Belleza                  | SAN         |
   | 68385 | Landázuri                   | SAN         |
   | 68397 | La Paz                      | SAN         |
   | 68406 | Lebrija                     | SAN         |
   | 68418 | Los Santos                  | SAN         |
   | 68425 | Macaravita                  | SAN         |
   | 68432 | Málaga                      | SAN         |
   | 68444 | Matanza                     | SAN         |
   | 68464 | Mogotes                     | SAN         |
   | 68468 | Molagavita                  | SAN         |
   | 68498 | Ocamonte                    | SAN         |
   | 68500 | Oiba                        | SAN         |
   | 68502 | Onzaga                      | SAN         |
   | 68522 | Palmar                      | SAN         |
   | 68524 | Palmas del Socorro          | SAN         |
   | 68533 | Páramo                      | SAN         |
   | 68547 | Piedecuesta                 | SAN         |
   | 68549 | Pinchote                    | SAN         |
   | 68572 | Puente Nacional             | SAN         |
   | 68573 | Puerto Parra                | SAN         |
   | 68575 | Puerto Wilches              | SAN         |
   | 68615 | Rionegro                    | SAN         |
   | 68655 | Sabana de Torres            | SAN         |
   | 68669 | San Andrés                  | SAN         |
   | 68673 | San Benito                  | SAN         |
   | 68679 | San Gil                     | SAN         |
   | 68682 | San Joaquín                 | SAN         |
   | 68684 | San José de Miranda         | SAN         |
   | 68686 | San Miguel                  | SAN         |
   | 68689 | San Vicente de Chucurí      | SAN         |
   | 68705 | Santa Bárbara               | SAN         |
   | 68720 | Santa Helena del Opón       | SAN         |
   | 68745 | Simacota                    | SAN         |
   | 68755 | Socorro                     | SAN         |
   | 68770 | Suaita                      | SAN         |
   | 68773 | Sucre                       | SAN         |
   | 68780 | Suratá                      | SAN         |
   | 68820 | Tona                        | SAN         |
   | 68855 | Valle de San José           | SAN         |
   | 68861 | Vélez                       | SAN         |
   | 68867 | Vetas                       | SAN         |
   | 68872 | Villanueva                  | SAN         |
   | 68895 | Zapatoca                    | SAN         |
   | 70001 | Sincelejo                   | SUC         |
   | 70110 | Buenavista                  | SUC         |
   | 70124 | Caimito                     | SUC         |
   | 70204 | Colosó                      | SUC         |
   | 70215 | Corozal                     | SUC         |
   | 70230 | Chalán                      | SUC         |
   | 70233 | El Roble                    | SUC         |
   | 70235 | Galeras                     | SUC         |
   | 70265 | Guaranda                    | SUC         |
   | 70400 | La Unión                    | SUC         |
   | 70418 | Los Palmitos                | SUC         |
   | 70429 | Majagual                    | SUC         |
   | 70473 | Morroa                      | SUC         |
   | 70508 | Ovejas                      | SUC         |
   | 70523 | Palmito                     | SUC         |
   | 70670 | Sampués                     | SUC         |
   | 70678 | San Benito Abad             | SUC         |
   | 70702 | San Juan de Betulia         | SUC         |
   | 70708 | San Marcos                  | SUC         |
   | 70713 | San Onofre                  | SUC         |
   | 70717 | San Pedro                   | SUC         |
   | 70742 | Sincé                       | SUC         |
   | 70771 | Sucre                       | SUC         |
   | 70820 | Santiago de Tolú            | SUC         |
   | 70823 | Toluviejo                   | SUC         |
   | 73001 | Ibagué                      | TOL         |
   | 73024 | Alpujarra                   | TOL         |
   | 73026 | Alvarado                    | TOL         |
   | 73030 | Ambalema                    | TOL         |
   | 73043 | Anzoátegui                  | TOL         |
   | 73055 | Armero                      | TOL         |
   | 73067 | Ataco                       | TOL         |
   | 73124 | Cajamarca                   | TOL         |
   | 73148 | Carmen de Apicalá           | TOL         |
   | 73152 | Casabianca                  | TOL         |
   | 73168 | Chaparral                   | TOL         |
   | 73200 | Coello                      | TOL         |
   | 73217 | Coyaima                     | TOL         |
   | 73226 | Cunday                      | TOL         |
   | 73236 | Dolores                     | TOL         |
   | 73268 | Espinal                     | TOL         |
   | 73270 | Falan                       | TOL         |
   | 73275 | Flandes                     | TOL         |
   | 73283 | Fresno                      | TOL         |
   | 73319 | Guamo                       | TOL         |
   | 73347 | Herveo                      | TOL         |
   | 73349 | Honda                       | TOL         |
   | 73352 | Icononzo                    | TOL         |
   | 73408 | Lérida                      | TOL         |
   | 73411 | Líbano                      | TOL         |
   | 73443 | Mariquita                   | TOL         |
   | 73449 | Melgar                      | TOL         |
   | 73461 | Murillo                     | TOL         |
   | 73483 | Natagaima                   | TOL         |
   | 73504 | Ortega                      | TOL         |
   | 73520 | Palocabildo                 | TOL         |
   | 73547 | Piedras                     | TOL         |
   | 73555 | Planadas                    | TOL         |
   | 73563 | Prado                       | TOL         |
   | 73585 | Purificación                | TOL         |
   | 73616 | Rioblanco                   | TOL         |
   | 73622 | Roncesvalles                | TOL         |
   | 73624 | Rovira                      | TOL         |
   | 73671 | Saldaña                     | TOL         |
   | 73675 | San Antonio                 | TOL         |
   | 73678 | San Luis                    | TOL         |
   | 73686 | Santa Isabel                | TOL         |
   | 73770 | Suárez                      | TOL         |
   | 73854 | Valle de San Juan           | TOL         |
   | 73861 | Venadillo                   | TOL         |
   | 73870 | Villahermosa                | TOL         |
   | 73873 | Villarrica                  | TOL         |
   | 76001 | Cali                        | VAC         |
   | 76020 | Alcalá                      | VAC         |
   | 76036 | Andalucía                   | VAC         |
   | 76041 | Ansermanuevo                | VAC         |
   | 76054 | Argelia                     | VAC         |
   | 76100 | Bolívar                     | VAC         |
   | 76109 | Buenaventura                | VAC         |
   | 76111 | Guadalajara de Buga         | VAC         |
   | 76113 | Bugalagrande                | VAC         |
   | 76122 | Caicedonia                  | VAC         |
   | 76126 | Calima                      | VAC         |
   | 76130 | Candelaria                  | VAC         |
   | 76147 | Cartago                     | VAC         |
   | 76233 | Dagua                       | VAC         |
   | 76243 | El Águila                   | VAC         |
   | 76246 | El Cairo                    | VAC         |
   | 76248 | El Cerrito                  | VAC         |
   | 76250 | El Dovio                    | VAC         |
   | 76275 | Florida                     | VAC         |
   | 76306 | Ginebra                     | VAC         |
   | 76318 | Guacarí                     | VAC         |
   | 76364 | Jamundí                     | VAC         |
   | 76377 | La Cumbre                   | VAC         |
   | 76400 | La Unión                    | VAC         |
   | 76403 | La Victoria                 | VAC         |
   | 76497 | Obando                      | VAC         |
   | 76520 | Palmira                     | VAC         |
   | 76563 | Pradera                     | VAC         |
   | 76606 | Restrepo                    | VAC         |
   | 76616 | Riofrío                     | VAC         |
   | 76622 | Roldanillo                  | VAC         |
   | 76670 | San Pedro                   | VAC         |
   | 76736 | Sevilla                     | VAC         |
   | 76823 | Toro                        | VAC         |
   | 76828 | Trujillo                    | VAC         |
   | 76834 | Tuluá                       | VAC         |
   | 76845 | Ulloa                       | VAC         |
   | 76863 | Versalles                   | VAC         |
   | 76869 | Vijes                       | VAC         |
   | 76890 | Yotoco                      | VAC         |
   | 76892 | Yumbo                       | VAC         |
   | 76895 | Zarzal                      | VAC         |
   | 81001 | Arauca                      | ARA         |
   | 81065 | Arauquita                   | ARA         |
   | 81220 | Cravo Norte                 | ARA         |
   | 81300 | Fortul                      | ARA         |
   | 81591 | Puerto Rondón               | ARA         |
   | 81736 | Saravena                    | ARA         |
   | 81794 | Tame                        | ARA         |
   | 85001 | Yopal                       | CAS         |
   | 85010 | Aguazul                     | CAS         |
   | 85015 | Chámeza                     | CAS         |
   | 85125 | Hato Corozal                | CAS         |
   | 85136 | La Salina                   | CAS         |
   | 85139 | Maní                        | CAS         |
   | 85162 | Monterrey                   | CAS         |
   | 85225 | Nunchía                     | CAS         |
   | 85230 | Orocué                      | CAS         |
   | 85250 | Paz de Ariporo              | CAS         |
   | 85263 | Pore                        | CAS         |
   | 85279 | Recetor                     | CAS         |
   | 85300 | Sabanalarga                 | CAS         |
   | 85315 | Sácama                      | CAS         |
   | 85325 | San Luis de Palenque        | CAS         |
   | 85400 | Támara                      | CAS         |
   | 85410 | Tauramena                   | CAS         |
   | 85430 | Trinidad                    | CAS         |
   | 85440 | Villanueva                  | CAS         |
   | 86001 | Mocoa                       | PUT         |
   | 86219 | Colón                       | PUT         |
   | 86320 | Orito                       | PUT         |
   | 86568 | Puerto Asís                 | PUT         |
   | 86569 | Puerto Caicedo              | PUT         |
   | 86571 | Puerto Guzmán               | PUT         |
   | 86573 | Leguízamo                   | PUT         |
   | 86749 | Sibundoy                    | PUT         |
   | 86755 | San Francisco               | PUT         |
   | 86757 | San Miguel                  | PUT         |
   | 86760 | Santiago                    | PUT         |
   | 86865 | Valle del Guamuez           | PUT         |
   | 86885 | Villagarzón                 | PUT         |
   | 88001 | San Andrés                  | SAP         |
   | 88564 | Providencia                 | SAP         |
   | 91001 | Leticia                     | AMA         |
   | 91263 | El Encanto                  | AMA         |
   | 91405 | La Chorrera                 | AMA         |
   | 91407 | La Pedrera                  | AMA         |
   | 91430 | La Victoria                 | AMA         |
   | 91460 | Mirití - Paraná             | AMA         |
   | 91530 | Puerto Alegría              | AMA         |
   | 91536 | Puerto Arica                | AMA         |
   | 91540 | Puerto Nariño               | AMA         |
   | 91669 | Puerto Santander            | AMA         |
   | 91798 | Tarapacá                    | AMA         |
   | 94001 | Inírida                     | GUA         |
   | 94343 | Barranco Minas              | GUA         |
   | 94663 | Mapiripana                  | GUA         |
   | 94883 | San Felipe                  | GUA         |
   | 94884 | Puerto Colombia             | GUA         |
   | 94885 | La Guadalupe                | GUA         |
   | 94886 | Cacahual                    | GUA         |
   | 94887 | Pana Pana                   | GUA         |
   | 94888 | Morichal                    | GUA         |
   | 95001 | San José del Guaviare       | GUV         |
   | 95015 | Calamar                     | GUV         |
   | 95025 | El Retorno                  | GUV         |
   | 95200 | Miraflores                  | GUV         |
   | 97001 | Mitú                        | VAU         |
   | 97161 | Carurú                      | VAU         |
   | 97511 | Pacoa                       | VAU         |
   | 97666 | Taraira                     | VAU         |
   | 97777 | Papunaua                    | VAU         |
   | 97889 | Yavaraté                    | VAU         |
   | 99001 | Puerto Carreño              | VID         |
   | 99524 | La Primavera                | VID         |
   | 99624 | Santa Rosalía               | VID         |
   | 99773 | Cumaribo                    | VID         |
   +-------+-----------------------------+-------------+
   ```

8. Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.

   ```
   SELECT *
   FROM products p
   WHERE NOT EXISTS (
     SELECT 1 FROM rates r WHERE r.poll_id IS NOT NULL AND EXISTS (
       SELECT 1 FROM companyproducts cp WHERE cp.product_id = p.id
     )
   ) AND NOT EXISTS (
       SELECT 1 FROM quality_products qp WHERE qp.product_id = p.id
     );
   +-----+--------------------------+--------------------------------------------------------+-------+-------------+-----------------------+
   | id  | name                     | detail                                                 | price | category_id | image                 |
   +-----+--------------------------+--------------------------------------------------------+-------+-------------+-----------------------+
   |  11 | Disco Duro Externo 1TB   | Almacenamiento portátil USB 3.0                        |   100 |           1 | disco_duro.jpg        |
   |  12 | Smartwatch S5            | Reloj inteligente con monitor de frecuencia cardíaca   |   250 |           1 | smartwatch_s5.jpg     |
   |  13 | Cargador Portátil        | Powerbank de 10,000 mAh                                |    50 |           1 | cargador_portatil.jpg |
   |  14 | Cable HDMI 2m            | Cable para transmisión de video Full HD                |    15 |           1 | cable_hdmi.jpg        |
   |  15 | Mousepad Gamer           | Mousepad grande con superficie antideslizante          |    20 |           1 | mousepad_gamer.jpg    |
   |  16 | Soporte Laptop           | Soporte ergonómico para laptop                         |    40 |           1 | soporte_laptop.jpg    |
   |  17 | Altavoz Bluetooth        | Altavoz portátil resistente al agua                    |    90 |           1 | altavoz_bluetooth.jpg |
   |  18 | Memoria USB 64GB         | Pendrive rápido USB 3.0                                |    25 |           1 | memoria_usb.jpg       |
   |  19 | Micrófono USB            | Micrófono para streaming y podcast                     |    75 |           1 | microfono_usb.jpg     |
   |  20 | Estabilizador de Voltaje | Protector contra picos de energía                      |    60 |           1 | estabilizador.jpg     |
   |  21 | Proyector HD             | Proyector portátil para presentaciones                 |   350 |           1 | proyector_hd.jpg      |
   | 111 | Producto 111             | Descripción producto 111                               |  9.99 |           1 | prod111.jpg           |
   | 112 | Producto 112             | Descripción producto 112                               |  14.5 |           2 | NULL                  |
   | 113 | Producto 113             | Detalle producto 113                                   |  8.75 |           3 | prod113.jpg           |
   | 114 | Producto 114             | Producto sin unidad medida                             |     7 |           1 | NULL                  |
   | 115 | Producto 115             | Producto 115                                           |  16.2 |           2 | prod115.jpg           |
   | 116 | Producto 116             | Detalle 116                                            |  11.3 |           3 | NULL                  |
   | 117 | Producto 117             | Producto especial                                      | 13.45 |           1 | prod117.jpg           |
   | 118 | Producto 118             | Detalle del producto 118                               |    10 |           2 | NULL                  |
   | 119 | Producto 119             | Producto 119 detalle                                   | 15.75 |           3 | prod119.jpg           |
   | 120 | Producto 120             | Producto final                                         |  12.6 |           1 | NULL                  |
   | 121 | Producto 121             | Detalle 121                                            |  14.1 |           2 | prod121.jpg           |
   | 122 | Producto 122             | Producto 122                                           |  9.95 |           3 | NULL                  |
   | 123 | Producto 123             | Detalle 123                                            |   7.8 |           1 | prod123.jpg           |
   | 124 | Producto 124             | Detalle 124                                            |   8.5 |           2 | NULL                  |
   | 125 | Producto 125             | Producto 125                                           | 10.25 |           3 | prod125.jpg           |
   | 126 | Producto 126             | Detalle producto 126                                   |  13.7 |           1 | NULL                  |
   | 127 | Producto 127             | Producto 127                                           | 11.85 |           2 | prod127.jpg           |
   | 128 | Producto 128             | Detalle 128                                            |  14.4 |           3 | NULL                  |
   | 129 | Producto 129             | Producto especial                                      |    12 |           1 | prod129.jpg           |
   | 130 | Producto 130             | Detalle final                                          |  16.1 |           2 | NULL                  |
   | 201 | Producto X               | Detalle Producto X                                     |   100 |           1 | NULL                  |
   | 202 | Producto X               | Detalle Producto X duplicado                           |   110 |           1 | NULL                  |
   | 203 | Producto Y               | Detalle Producto Y                                     |   150 |           1 | NULL                  |
   | 999 | Producto sin calificar   | Este producto es favorito pero no tiene calificaciones | 19.99 |           1 | NULL                  |
   +-----+--------------------------+--------------------------------------------------------+-------+-------------+-----------------------+
   ```

9. Como auditor, quiero listar los beneficios que no están asignados a ninguna audiencia.

   ```
   SELECT b.description
   FROM benefits b
   WHERE NOT EXISTS (
       SELECT 1 FROM audiencebenefits ab WHERE ab.benefit_id = b.id
   );
   +--------------------+
   | description        |
   +--------------------+
   | Acceso exclusivo   |
   | Garantía Extendida |
   +--------------------+
   ```

10. Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.

    ```
    SELECT DISTINCT 
      p.name
    FROM products p
    JOIN details_favorites df ON p.id = df.product_id
    JOIN favorites f ON df.favorite_id = f.id
    WHERE f.customer_id = 1
      AND NOT EXISTS (
        SELECT 1 
        FROM companyproducts cp 
        WHERE cp.product_id = p.id
      );
    +-----------+
    | name      |
    +-----------+
    | Mouse Pro |
    +-----------+
    ```

11. Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.

    ```
    SELECT 
      p.id,
      p.name
    FROM products p
    JOIN companyproducts cp ON p.id = cp.product_id
    JOIN companies c ON cp.company_id = c.id
    WHERE c.city_id IN (
        SELECT cm.code
        FROM citiesormunicipalities cm
        JOIN companies c ON cm.code = c.city_id
        GROUP BY cm.code
        HAVING COUNT(c.id) <= 3
    );
    +----+------------------------+
    | id | name                   |
    +----+------------------------+
    |  1 | Laptop XYZ             |
    |  8 | Impresora Láser        |
    |  9 | Cámara Web HD          |
    | 10 | Router WiFi            |
    | 11 | Disco Duro Externo 1TB |
    | 12 | Smartwatch S5          |
    | 13 | Cargador Portátil      |
    | 14 | Cable HDMI 2m          |
    | 15 | Mousepad Gamer         |
    | 16 | Soporte Laptop         |
    +----+------------------------+
    ```

12. Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.

    ```
    WITH avg_quality AS (
      SELECT AVG(rating) AS avg_all
      FROM quality_products
    )
    SELECT p.*, qp.rating
    FROM products p
    JOIN quality_products qp ON p.id = qp.product_id
    JOIN avg_quality aq
    WHERE qp.rating > aq.avg_all;
    +----+------------------+-------------------------------------------------------+-------+-------------+----------------------+--------+
    | id | name             | detail                                                | price | category_id | image                | rating |
    +----+------------------+-------------------------------------------------------+-------+-------------+----------------------+--------+
    |  1 | Laptop XYZ       | Laptop para trabajo                                   |  2500 |           1 | laptop.jpg           |   4.50 |
    |  1 | Laptop XYZ       | Laptop para trabajo                                   |  2500 |           1 | laptop.jpg           |   4.30 |
    |  2 | Mouse Pro        | Mouse ergonómico                                      |   150 |           1 | mouse.jpg            |   4.70 |
    |  2 | Mouse Pro        | Mouse ergonómico                                      |   150 |           1 | mouse.jpg            |   4.40 |
    |  3 | Smartphone Alpha | Teléfono inteligente con cámara de 48MP               |  1200 |           1 | smartphone_alpha.jpg |   4.70 |
    |  7 | Tablet Z10       | Tablet con pantalla de 10 pulgadas                    |   600 |           1 | tablet_z10.jpg       |   4.90 |
    |  7 | Tablet Z10       | Tablet con pantalla de 10 pulgadas                    |   600 |           1 | tablet_z10.jpg       |   4.80 |
    |  8 | Impresora Láser  | Impresora monocromo de alta velocidad                 |   450 |           1 | impresora_laser.jpg  |   4.30 |
    |  9 | Cámara Web HD    | Cámara para videoconferencias con micrófono integrado |   120 |           1 | camara_web.jpg       |   4.60 |
    | 10 | Router WiFi      | Router inalámbrico de doble banda                     |   130 |           1 | router_wifi.jpg      |   4.60 |
    +----+------------------+-------------------------------------------------------+-------+-------------+----------------------+--------+
    ```

13. Como gestor, quiero ver empresas que sólo venden productos de una única categoría.

    ```
    SELECT c.id, c.name
    FROM companies c
    JOIN companyproducts cp ON c.id = cp.company_id
    JOIN products p ON cp.product_id = p.id
    GROUP BY c.id, c.name
    HAVING COUNT(DISTINCT p.category_id) = 1;
    +-----------+--------------+
    | id        | name         |
    +-----------+--------------+
    | 900000001 | TechCorp     |
    | 900654321 | DigitalHouse |
    | 900987654 | SmartTech    |
    +-----------+--------------+
    ```

14. Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.

    ```
    SELECT 
      cp.product_id, 
      MAX(cp.price) AS max_price
    FROM companyproducts cp
    GROUP BY cp.product_id
    ORDER BY max_price DESC
    LIMIT 10;
    +------------+-----------+
    | product_id | max_price |
    +------------+-----------+
    |          1 |   2550.00 |
    |          3 |   1150.00 |
    |          7 |    580.00 |
    |          8 |    440.00 |
    |          4 |    330.00 |
    |         12 |    245.00 |
    |          5 |    190.00 |
    |          6 |    170.00 |
    |        203 |    150.00 |
    |         10 |    125.00 |
    +------------+-----------+
    ```

15. Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.

    ```
    SELECT DISTINCT p.name
    FROM products p
    JOIN details_favorites df ON p.id = df.product_id
    WHERE df.favorite_id IN (
        SELECT f.id
        FROM favorites f
    )
    AND p.id IN (
        SELECT qp.product_id
        FROM quality_products qp
        WHERE qp.rating > 4
    );
    +------------+
    | name       |
    +------------+
    | Laptop XYZ |
    | Mouse Pro  |
    +------------+
    ```

16. Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.

    ```
    SELECT p.id, p.name
    FROM products p
    WHERE p.image IS NULL
      AND EXISTS (
        SELECT 1 FROM quality_products qp
        WHERE qp.product_id = p.id AND qp.rating > 4
      );
    +-----+--------------+
    | id  | name         |
    +-----+--------------+
    | 112 | Producto 112 |
    | 114 | Producto 114 |
    | 116 | Producto 116 |
    +-----+--------------+
    ```

17. Como auditor, quiero ver los planes de membresía sin periodo vigente.

    ```
    SELECT m.id, m.name
    FROM memberships m
    LEFT JOIN membershipperiods mp ON m.id = mp.membership_id
    WHERE mp.period_id IS NULL;
    +----+-----------------------+
    | id | name                  |
    +----+-----------------------+
    |  1 | Membresía Básica      |
    |  2 | Membresía Premium     |
    |  3 | Membresía VIP         |
    |  4 | Membresía Corporativa |
    +----+-----------------------+
    ```

18. Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.

    ```
    SELECT 
      b.id, 
      b.description, 
      COUNT(DISTINCT ba.audience_id) AS num_audiences 
    FROM benefits b
    JOIN audiencebenefits ba ON b.id = ba.benefit_id 
    GROUP BY b.id, b.description 
    HAVING COUNT(DISTINCT ba.audience_id) > 1;
    +-----+------------------------------+---------------+
    | id  | description                  | num_audiences |
    +-----+------------------------------+---------------+
    | 101 | Acceso a contenido exclusivo |             2 |
    | 102 | Descuento en eventos         |             2 |
    | 103 | Soporte prioritario          |             2 |
    +-----+------------------------------+---------------+
    ```

19. Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.

    ```
    SELECT DISTINCT 
      c.id, 
      c.name
    FROM companies c
    JOIN companyproducts cp ON c.id = cp.company_id
    LEFT JOIN unitofmeasure u ON cp.unitmeasure_id = u.id
    WHERE u.id IS NULL;
    +------+--------------+
    | id   | name         |
    +------+--------------+
    | C001 | Empresa Alfa |
    | C002 | Empresa Beta |
    +------+--------------+
    ```

20. Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.

    ```
    SELECT c.id, c.name
    FROM customers c
    JOIN customer_memberships cm ON c.id = cm.customer_id
    WHERE cm.status = 'active'
      AND NOT EXISTS (
        SELECT 1
        FROM favorites f
        WHERE f.customer_id = c.id
      );
    +----+-----------------------+
    | id | name                  |
    +----+-----------------------+
    | 21 | Cliente Sin Favoritos |
    +----+-----------------------+
    ```
