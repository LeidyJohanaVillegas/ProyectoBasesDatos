🔹 **3. Funciones Agregadas**

### **1. Obtener el promedio de calificación por producto**

> *"Como analista, quiero obtener el promedio de calificación por producto."*

🔍 **Explicación para dummies:** La persona encargada de revisar el rendimiento quiere saber **qué tan bien calificado está cada producto**. Con `AVG(rating)` agrupado por `product_id`, puede verlo de forma resumida.

```
SELECT 
  product_id,
  AVG(rating) AS average_rating
FROM 
  quality_products
GROUP BY 
  product_id;
+------------+----------------+
| product_id | average_rating |
+------------+----------------+
|          1 |       4.400000 |
|          2 |       4.550000 |
|          3 |       4.450000 |
|          4 |       4.100000 |
|          5 |       3.850000 |
|          6 |       4.100000 |
|          7 |       4.850000 |
|          8 |       4.150000 |
|          9 |       4.150000 |
|         10 |       4.250000 |
|        112 |       4.300000 |
|        114 |       4.350000 |
|        116 |       4.700000 |
+------------+----------------+
```

------

### **2. Contar cuántos productos ha calificado cada cliente**

> *"Como gerente, desea contar cuántos productos ha calificado cada cliente."*

🔍 **Explicación:** Aquí se quiere saber **quiénes están activos opinando**. Se usa `COUNT(*)` sobre `rates`, agrupando por `customer_id`.

```
SELECT
  c.id AS customer_id,
  c.name AS customer_name,
  COUNT(r.customer_id) AS total_rated_products
FROM
  customers c
LEFT JOIN
  rates r ON c.id = r.customer_id
GROUP BY
  c.id, c.name;
+-------------+-----------------------+----------------------+
| customer_id | customer_name         | total_rated_products |
+-------------+-----------------------+----------------------+
|           1 | Carlos Pérez          |                    2 |
|           2 | Ana Gómez             |                    1 |
|           3 | Luis Torres           |                    1 |
|           4 | María Rivas           |                    1 |
|           5 | Diego Mendoza         |                    0 |
|           6 | Sofía Castillo        |                    0 |
|           7 | Andrés Molina         |                    0 |
|           8 | Luisa Fernández       |                    0 |
|           9 | Pedro López           |                    0 |
|          10 | Carolina Reyes        |                    0 |
|          21 | Cliente Sin Favoritos |                    0 |
+-------------+-----------------------+----------------------+
```

------

### **3. Sumar el total de beneficios asignados por audiencia**

> *"Como auditor, quiere sumar el total de beneficios asignados por audiencia."*

🔍 **Explicación:** El auditor busca **cuántos beneficios tiene cada tipo de usuario**. Con `COUNT(*)` agrupado por `audience_id` en `audiencebenefits`, lo obtiene.

```
SELECT
  audience_id,
  COUNT(*) AS total_benefits
FROM
  audiencebenefits
GROUP BY
  audience_id;
+-------------+----------------+
| audience_id | total_benefits |
+-------------+----------------+
|           1 |              3 |
|           2 |              3 |
|           3 |              1 |
|           4 |              1 |
|           5 |              2 |
+-------------+----------------+
```

------

### **4. Calcular la media de productos por empresa**

> *"Como administrador, desea conocer la media de productos por empresa."*

🔍 **Explicación:** El administrador quiere saber si **las empresas están ofreciendo pocos o muchos productos**. Cuenta los productos por empresa y saca el promedio con `AVG(cantidad)`.

```
SELECT 
  AVG(product_count) AS average_products_per_company
FROM (
  SELECT 
    company_id,
    COUNT(*) AS product_count
  FROM 
    companyproducts
  GROUP BY 
    company_id
) AS company_totals;
+------------------------------+
| average_products_per_company |
+------------------------------+
|                       3.6923 |
+------------------------------+
```

------

### **5. Contar el total de empresas por ciudad**

> *"Como supervisor, quiere ver el total de empresas por ciudad."*

🔍 **Explicación:** La idea es ver **en qué ciudades hay más movimiento empresarial**. Se usa `COUNT(*)` en `companies`, agrupando por `city_id`.

```
SELECT 
 city_id,
 COUNT(*) AS total_companies
FROM 
 companies
GROUP BY 
 city_id
ORDER BY 
 total_companies DESC;
+---------+-----------------+
| city_id | total_companies |
+---------+-----------------+
| 05001   |              30 |
| NULL    |               3 |
| 08001   |               3 |
+---------+-----------------+
```

------

### **6. Calcular el promedio de precios por unidad de medida**

> *"Como técnico, desea obtener el promedio de precios de productos por unidad de medida."*

🔍 **Explicación:** Se necesita saber si **los precios son coherentes según el tipo de medida**. Con `AVG(price)` agrupado por `unit_id`, se compara cuánto cuesta el litro, kilo, unidad, etc.

```
SELECT 
  u.id AS unit_id,
  u.description AS unit_name, 
  AVG(cp.price) AS average_price
FROM 
  companyproducts cp
LEFT JOIN 
  unitofmeasure u ON cp.unitmeasure_id = u.id
GROUP BY 
  u.id, u.description 
ORDER BY 
  average_price DESC;
  +---------+-----------+---------------+
| unit_id | unit_name | average_price |
+---------+-----------+---------------+
|       1 | Unidad    |    253.675769 |
|       2 | Kilogramo |     14.212857 |
|       3 | Litro     |     11.348000 |
|    NULL | NULL      |     10.154000 |
+---------+-----------+---------------+
```

------

### **7. Contar cuántos clientes hay por ciudad**

> *"Como gerente, quiere ver el número de clientes registrados por cada ciudad."*

🔍 **Explicación:** Con `COUNT(*)` agrupado por `city_id` en la tabla `customers`, se obtiene **la cantidad de clientes que hay en cada zona**.

```
SELECT 
  city_id,
  COUNT(*) AS total_customers
FROM 
  customers
GROUP BY 
  city_id
ORDER BY 
  total_customers DESC;
+---------+-----------------+
| city_id | total_customers |
+---------+-----------------+
| 08001   |               3 |
| 08078   |               2 |
| 08137   |               2 |
| 08141   |               2 |
| 08296   |               2 |
+---------+-----------------+
```

------

### **8. Calcular planes de membresía por periodo**

> *"Como operador, desea contar cuántos planes de membresía existen por periodo."*

🔍 **Explicación:** Sirve para ver **qué tantos planes están vigentes cada mes o trimestre**. Se agrupa por periodo (`start_date`, `end_date`) y se cuenta cuántos registros hay.

```
SELECT 
  YEAR(start_date) AS year,
  MONTH(start_date) AS month,
  COUNT(*) AS total_plans
FROM 
  customer_memberships
GROUP BY 
  YEAR(start_date), MONTH(start_date)
ORDER BY 
  year, month;
+------+-------+-------------+
| year | month | total_plans |
+------+-------+-------------+
| 2024 |     1 |           1 |
| 2025 |     1 |           2 |
| 2025 |     2 |           1 |
+------+-------+-------------+
```

------

### **9. Ver el promedio de calificaciones dadas por un cliente a sus favoritos**

> *"Como cliente, quiere ver el promedio de calificaciones que ha otorgado a sus productos favoritos."*

🔍 **Explicación:** El cliente quiere saber **cómo ha calificado lo que más le gusta**. Se hace un `JOIN` entre favoritos y calificaciones, y se saca `AVG(rating)`.

```
SELECT
  f.customer_id,
  c.name AS customer_name,
  IFNULL(AVG(r.rating), 0) AS avg_rating_for_favorites
FROM
  favorites f
LEFT JOIN
  rates r ON f.customer_id = r.customer_id AND f.company_id = r.company_id
JOIN
  customers c ON f.customer_id = c.id
GROUP BY
  f.customer_id, c.name;
+-------------+-----------------+--------------------------+
| customer_id | customer_name   | avg_rating_for_favorites |
+-------------+-----------------+--------------------------+
|           1 | Carlos Pérez    |                 0.000000 |
|           2 | Ana Gómez       |                 0.000000 |
|           3 | Luis Torres     |                 0.000000 |
|           4 | María Rivas     |                 0.000000 |
|           5 | Diego Mendoza   |                 0.000000 |
|           6 | Sofía Castillo  |                 0.000000 |
|           7 | Andrés Molina   |                 0.000000 |
|           8 | Luisa Fernández |                 0.000000 |
|           9 | Pedro López     |                 0.000000 |
|          10 | Carolina Reyes  |                 0.000000 |
+-------------+-----------------+--------------------------+
SELECT
  f.customer_id,
  c.name AS customer_name,
  AVG(r.rating) AS avg_rating_for_favorites
FROM
  favorites f
LEFT JOIN
  rates r ON f.customer_id = r.customer_id AND f.company_id = r.company_id
JOIN
  customers c ON f.customer_id = c.id
GROUP BY
  f.customer_id, c.name;
+-------------+-----------------+--------------------------+
| customer_id | customer_name   | avg_rating_for_favorites |
+-------------+-----------------+--------------------------+
|           1 | Carlos Pérez    |                     NULL |
|           2 | Ana Gómez       |                     NULL |
|           3 | Luis Torres     |                     NULL |
|           4 | María Rivas     |                     NULL |
|           5 | Diego Mendoza   |                     NULL |
|           6 | Sofía Castillo  |                     NULL |
|           7 | Andrés Molina   |                     NULL |
|           8 | Luisa Fernández |                     NULL |
|           9 | Pedro López     |                     NULL |
|          10 | Carolina Reyes  |                     NULL |
+-------------+-----------------+--------------------------+
```

------

### **10. Consultar la fecha más reciente en que se calificó un producto**

> *"Como auditor, desea obtener la fecha más reciente en la que se calificó un producto."*

🔍 **Explicación:** Busca el `MAX(created_at)` agrupado por producto. Así sabe **cuál fue la última vez que se evaluó cada uno**.

```
SELECT
  r.poll_id AS product_id,
  MAX(r.daterating) AS last_rating_date
FROM
  rates r
GROUP BY
  r.poll_id;
+------------+---------------------+
| product_id | last_rating_date    |
+------------+---------------------+
|          1 | 2025-07-16 17:17:23 |
+------------+---------------------+
```

------

### **11. Obtener la desviación estándar de precios por categoría**

> *"Como desarrollador, quiere conocer la variación de precios por categoría de producto."*

🔍 **Explicación:** Usando `STDDEV(price)` en `companyproducts` agrupado por `category_id`, se puede ver **si hay mucha diferencia de precios dentro de una categoría**.

```
SELECT 
 c.id,
 c.description, 
 STDDEV(cp.price) AS price_variation
FROM categories c
JOIN products p ON c.id = p.category_id
JOIN companyproducts cp ON p.id = cp.product_id
GROUP BY c.id, c.description;
+----+-------------+-------------------+
| id | description | price_variation   |
+----+-------------+-------------------+
|  1 | Electrónica | 511.6179628165367 |
|  2 | Alimentos   | 3.648753211714928 |
|  3 | Ropa        | 2.839773562960415 |
+----+-------------+-------------------+
```

------

### **12. Contar cuántas veces un producto fue favorito**

> *"Como técnico, desea contar cuántas veces un producto fue marcado como favorito."*

🔍 **Explicación:** Con `COUNT(*)` en `details_favorites`, agrupado por `product_id`, se obtiene **cuáles productos son los más populares entre los clientes**.

```
SELECT
  company_id,
  COUNT(*) AS times_favorited
FROM
  favorites
GROUP BY
  company_id;
+------------+-----------------+
| company_id | times_favorited |
+------------+-----------------+
| 900000001  |              15 |
| C001       |               6 |
| C002       |               3 |
| C003       |               4 |
| C004       |               2 |
| C005       |               3 |
| C006       |               2 |
| C007       |               1 |
| C008       |               1 |
+------------+-----------------+
```

------

### **13. Calcular el porcentaje de productos evaluados**

> *"Como director, quiere saber qué porcentaje de productos han sido calificados al menos una vez."*

🔍 **Explicación:** Cuenta cuántos productos hay en total y cuántos han sido evaluados (`rates`). Luego calcula `(evaluados / total) * 100`.

```
SELECT
  (COUNT(DISTINCT r.poll_id) / (SELECT COUNT(*) FROM products)) * 100 AS evaluated_percentage
FROM
  rates r;
+----------------------+
| evaluated_percentage |
+----------------------+
|               2.1277 |
+----------------------+
```

------

### **14. Ver el promedio de rating por encuesta**

> *"Como analista, desea conocer el promedio de rating por encuesta."*

🔍 **Explicación:** Agrupa por `poll_id` en `rates`, y calcula el `AVG(rating)` para ver **cómo se comportó cada encuesta**.

```
SELECT
  r.poll_id,
  AVG(r.rating) AS avg_rating_per_poll
FROM
  rates r
GROUP BY
  r.poll_id;
+---------+---------------------+
| poll_id | avg_rating_per_poll |
+---------+---------------------+
|       1 |            3.900000 |
+---------+---------------------+
```

------

### **15. Calcular el promedio y total de beneficios por plan**

> *"Como gestor, quiere obtener el promedio y el total de beneficios asignados a cada plan de membresía."*

🔍 **Explicación:** Agrupa por `membership_id` en `membershipbenefits`, y usa `COUNT(*)` y `AVG(beneficio)` si aplica (si hay ponderación).

```
SELECT
  mb.membership_id,
  COUNT(*) AS total_benefits
FROM
  membershipbenefits mb
GROUP BY
  mb.membership_id;
+---------------+----------------+
| membership_id | total_benefits |
+---------------+----------------+
|             1 |              2 |
|             2 |              1 |
|             4 |              2 |
+---------------+----------------+
```

------

### **16. Obtener media y varianza de precios por empresa**

> *"Como gerente, desea obtener la media y la varianza del precio de productos por empresa."*

🔍 **Explicación:** Se agrupa por `company_id` y se usa `AVG(price)` y `VARIANCE(price)` para saber **qué tan consistentes son los precios por empresa**.

```
SELECT
  company_id,
  AVG(price) AS avg_price,
  VARIANCE(price) AS price_variance
FROM
  companyproducts
GROUP BY
  company_id;
+------------+------------+--------------------+
| company_id | avg_price  | price_variance     |
+------------+------------+--------------------+
| 900000001  | 484.000000 |             132304 |
| 900654321  | 664.000000 |             905674 |
| 900987654  |  72.600000 |  7587.839999999999 |
| C001       |  49.258000 |        2088.203456 |
| C002       |  45.500000 |         3646.53125 |
| C003       |  13.913333 |  3.763355555555553 |
| C004       |  14.416667 | 12.097222222222221 |
| C005       |   9.746667 |  5.770022222222226 |
| C006       |  16.430000 | 27.758466666666664 |
| C007       |  15.416667 | 0.9305555555555554 |
| C008       |  14.346667 |  6.227355555555555 |
| C009       |   9.500000 |              0.875 |
| C010       |  13.580000 | 1.2544666666666677 |
+------------+------------+--------------------+
```

------

### **17. Ver total de productos disponibles en la ciudad del cliente**

> *"Como cliente, quiere ver cuántos productos están disponibles en su ciudad."*

🔍 **Explicación:** Hace un `JOIN` entre `companies`, `companyproducts` y `citiesormunicipalities`, filtrando por la ciudad del cliente. Luego se cuenta.

```
SELECT
  c.code AS city_code,
  COUNT(cp.product_id) AS total_products_in_city
FROM
  customers cu
JOIN
  citiesormunicipalities c ON cu.city_id = c.code  -- Ciudad del cliente
JOIN
  companies com ON c.code = com.city_id 
JOIN
  companyproducts cp ON com.id = cp.company_id  -- Productos de las empresas
WHERE
  cu.id = 1  -- Aquí se reemplaza '1' con el ID del cliente
GROUP BY
  c.code;
+-----------+------------------------+
| city_code | total_products_in_city |
+-----------+------------------------+
| 08001     |                     10 |
+-----------+------------------------+
```

------

### **18. Contar productos únicos por tipo de empresa**

> *"Como administrador, desea contar los productos únicos por tipo de empresa."*

🔍 **Explicación:** Agrupa por `company_type_id` y cuenta cuántos productos diferentes tiene cada tipo de empresa.

```
SELECT
  com.type_id AS company_type_id,
  COUNT(DISTINCT cp.product_id) AS unique_products_count
FROM
  companies com
JOIN
  companyproducts cp ON com.id = cp.company_id
GROUP BY
  com.type_id;
+-----------------+-----------------------+
| company_type_id | unique_products_count |
+-----------------+-----------------------+
|            NULL |                    12 |
|               1 |                    15 |
+-----------------+-----------------------+
```

------

### **19. Ver total de clientes sin correo electrónico registrado**

> *"Como operador, quiere saber cuántos clientes no han registrado su correo."*

🔍 **Explicación:** Filtra `customers WHERE email IS NULL` y hace un `COUNT(*)`. Esto ayuda a mejorar la base de datos para campañas.

```
SELECT
  COUNT(*) AS total_customers_without_email
FROM
  customers
WHERE
  email IS NULL;
+-------------------------------+
| total_customers_without_email |
+-------------------------------+
|                             0 |
+-------------------------------+
```

------

### **20. Empresa con más productos calificados**

> *"Como especialista, desea obtener la empresa con el mayor número de productos calificados."*

🔍 **Explicación:** Hace un `JOIN` entre `companies`, `companyproducts`, y `rates`, agrupa por empresa y usa `COUNT(DISTINCT product_id)`, ordenando en orden descendente y tomando solo el primero.

```
SELECT
  com.name AS company_name,
  COUNT(DISTINCT cp.product_id) AS total_rated_products
FROM
  companies com
JOIN
  companyproducts cp ON com.id = cp.company_id
JOIN
  rates r ON com.id = r.company_id
GROUP BY
  com.id
ORDER BY
  total_rated_products DESC
LIMIT 1;
+--------------+----------------------+
| company_name | total_rated_products |
+--------------+----------------------+
| Empresa Alfa |                    5 |
+--------------+----------------------+
```

