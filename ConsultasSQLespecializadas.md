## 🔹 **1. Consultas SQL Especializadas**

1. Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.

   ```
   SELECT 
     co.name AS empresa,
     p.name AS producto,
     MIN(cp.price) AS precio_minimo,
     city.name AS ciudad
   FROM companyproducts cp
   JOIN companies co ON cp.company_id = co.id
   JOIN products p ON cp.product_id = p.id
   JOIN citiesormunicipalities city ON co.city_id = city.code
   GROUP BY co.name, p.name, city.name
   ORDER BY precio_minimo DESC;
   +--------------+------------------------+---------------+--------------+
   | empresa      | producto               | precio_minimo | ciudad       |
   +--------------+------------------------+---------------+--------------+
   | DigitalHouse | Laptop XYZ             |       2550.00 | Barranquilla |
   | DigitalHouse | Impresora Láser        |        440.00 | Barranquilla |
   | SmartTech    | Smartwatch S5          |        245.00 | Barranquilla |
   | DigitalHouse | Router WiFi            |        125.00 | Barranquilla |
   | DigitalHouse | Cámara Web HD          |        110.00 | Barranquilla |
   | DigitalHouse | Disco Duro Externo 1TB |         95.00 | Barranquilla |
   | SmartTech    | Cargador Portátil      |         48.00 | Barranquilla |
   | SmartTech    | Soporte Laptop         |         38.00 | Barranquilla |
   | SmartTech    | Mousepad Gamer         |         18.00 | Barranquilla |
   | SmartTech    | Cable HDMI 2m          |         14.00 | Barranquilla |
   +--------------+------------------------+---------------+--------------+
   ```

2. Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.

   ```
   SELECT
     r.customer_id,
     cu.name,
     COUNT(*) AS ratings_count
   FROM quality_products r
   JOIN customers cu ON cu.id = r.customer_id
   WHERE r.daterating >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
   GROUP BY r.customer_id, cu.name
   ORDER BY ratings_count DESC
   LIMIT 5;
   +-------------+---------------+---------------+
   | customer_id | name          | ratings_count |
   +-------------+---------------+---------------+
   |           4 | María Rivas   |             4 |
   |           1 | Carlos Pérez  |             3 |
   |           2 | Ana Gómez     |             3 |
   |           3 | Luis Torres   |             3 |
   |           5 | Diego Mendoza |             2 |
   +-------------+---------------+---------------+
   ```

3. Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.

   ```
   SELECT 
     cat.description AS category,
     uom.description AS unit_of_measure,
     COUNT(*) AS product_count
   FROM companyproducts cp
   JOIN products p ON p.id = cp.product_id
   JOIN categories cat ON cat.id = p.category_id
   LEFT JOIN unitofmeasure uom ON uom.id = cp.unitmeasure_id
   GROUP BY cat.description, uom.description;
   +------------+-----------------+---------------+
   | category   | unit_of_measure | product_count |
   +------------+-----------------+---------------+
   | Tecnología | Unidad          |            15 |
   +------------+-----------------+---------------+
   ```

4. Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.

   ```
   SELECT DISTINCT 
     p.id, p.name,
     AVG(qp.rating) AS avg_rating
   FROM quality_products qp
   JOIN products p ON p.id = qp.product_id
   GROUP BY p.id, p.name
   HAVING avg_rating > (
     SELECT AVG(rating) FROM quality_products
   );
   +----+------------------+------------+
   | id | name             | avg_rating |
   +----+------------------+------------+
   |  1 | Laptop XYZ       |   4.400000 |
   |  2 | Mouse Pro        |   4.550000 |
   |  3 | Smartphone Alpha |   4.450000 |
   |  7 | Tablet Z10       |   4.850000 |
   +----+------------------+------------+
   ```

5. Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.

   ```
   SELECT 
     comp.id, 
     comp.name
   FROM companies comp
   LEFT JOIN rates r ON r.company_id = comp.id
   LEFT JOIN quality_products qp ON qp.company_id = comp.id
   WHERE r.company_id IS NULL AND qp.company_id IS NULL;
   +-----------+-----------------+
   | id        | name            |
   +-----------+-----------------+
   | 900000002 | ElectroHome     |
   | 900000003 | ModaPlus        |
   | 900000004 | SaludVida       |
   | 900000005 | DeportesYa      |
   | 900000006 | JuguetesMundo   |
   | 900000007 | AutoParts       |
   | 900000008 | LibroFácil      |
   | 900000009 | MúsicaViva      |
   | 900000010 | ToolKing        |
   | 900000011 | JardinNatural   |
   | 900000012 | MascotasFelices |
   | 900000013 | SuperMercado    |
   | 900000014 | OficinaExpress  |
   | 900000015 | ServiciosPro    |
   | 900000016 | GastroDelicia   |
   | 900000017 | ViajesPlus      |
   | 900000018 | ElectroPlus     |
   | 900000019 | SoftWareHouse   |
   | 900000020 | EducaSmart      |
   | 900000021 | ArteManía       |
   | 900000022 | FinanzasSeguras |
   | 900000023 | RealEstatePro   |
   | 900000024 | IndustriaMax    |
   | 900000025 | FotoStudio      |
   | 900000026 | GameWorld       |
   | 900000027 | MusiPro         |
   | 900000028 | ModaKids        |
   | 900000029 | SafeWatch       |
   | 900000030 | GreenEnergy     |
   | 900123456 | Ejemplo S.A.    |
   | 900654321 | DigitalHouse    |
   | 900987654 | SmartTech       |
   +-----------+-----------------+
   ```

6. Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.

   ```
   SELECT 
     p.id, p.name,
     COUNT(DISTINCT f.customer_id) AS num_customers
   FROM details_favorites df
   JOIN favorites f ON f.id = df.favorite_id
   JOIN products p ON p.id = df.product_id
   GROUP BY p.id, p.name
   HAVING num_customers > 10;
   +----+------------+---------------+
   | id | name       | num_customers |
   +----+------------+---------------+
   |  1 | Laptop XYZ |            11 |
   +----+------------+---------------+
   ```

7. Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.

   ```
   SELECT 
     ci.name AS city,
     cat.description AS category,
     COUNT(DISTINCT comp.id) AS active_companies
   FROM companies comp
   JOIN citiesormunicipalities ci ON ci.code = comp.city_id
   JOIN categories cat ON cat.id = comp.category_id
   GROUP BY ci.name, cat.description;
   +--------------+------------+------------------+
   | city         | category   | active_companies |
   +--------------+------------+------------------+
   | Barranquilla | Tecnología |                3 |
   +--------------+------------+------------------+
   ```

8. Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.

   ```
   SELECT 
     t.city, 
     t.product_id, 
     t.product_name, 
     t.avg_rating
   FROM (
     SELECT 
       ci.name AS city,
       p.id AS product_id, p.name AS product_name,
       AVG(qp.rating) AS avg_rating,
       ROW_NUMBER() OVER (PARTITION BY ci.code ORDER BY AVG(qp.rating) DESC) AS rn
     FROM quality_products qp
      JOIN customers cu ON cu.id = qp.customer_id
      JOIN citiesormunicipalities ci ON ci.code = cu.city_id
      JOIN products p ON p.id = qp.product_id
      GROUP BY ci.code, ci.name, p.id, p.name
   ) t
   WHERE t.rn <= 10;
   +------------------+------------+-----------------------+------------+
   | city             | product_id | product_name          | avg_rating |
   +------------------+------------+-----------------------+------------+
   | Barranquilla     |          2 | Mouse Pro             |   4.700000 |
   | Barranquilla     |          3 | Smartphone Alpha      |   4.700000 |
   | Barranquilla     |          1 | Laptop XYZ            |   4.500000 |
   | Barranquilla     |          8 | Impresora Láser       |   4.000000 |
   | Baranoa          |          3 | Smartphone Alpha      |   4.200000 |
   | Baranoa          |          4 | Monitor 24"           |   4.100000 |
   | Baranoa          |          9 | Cámara Web HD         |   3.700000 |
   | Campo de la Cruz |         10 | Router WiFi           |   4.600000 |
   | Campo de la Cruz |          6 | Auriculares Bluetooth |   4.100000 |
   | Campo de la Cruz |          5 | Teclado Mecánico      |   3.850000 |
   | Candelaria       |          7 | Tablet Z10            |   4.900000 |
   | Candelaria       |          9 | Cámara Web HD         |   4.600000 |
   | Candelaria       |          1 | Laptop XYZ            |   4.300000 |
   | Candelaria       |          8 | Impresora Láser       |   4.300000 |
   | Candelaria       |          6 | Auriculares Bluetooth |   4.100000 |
   | Galapa           |          7 | Tablet Z10            |   4.800000 |
   | Galapa           |          2 | Mouse Pro             |   4.400000 |
   | Galapa           |         10 | Router WiFi           |   3.900000 |
   +------------------+------------+-----------------------+------------+
   ```

9. Como técnico, quiero identificar productos sin unidad de medida asignada.

   ```
   SELECT DISTINCT
     p.id, 
     p.name
   FROM companyproducts cp
   JOIN products p ON p.id = cp.product_id
   WHERE cp.unitmeasure_id IS NULL;
   +-----+--------------+
   | id  | name         |
   +-----+--------------+
   | 114 | Producto 114 |
   | 115 | Producto 115 |
   | 119 | Producto 119 |
   | 123 | Producto 123 |
   | 126 | Producto 126 |
   | 129 | Producto 129 |
   +-----+--------------+
   ```

10. Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.
   

        SELECT 
          m.id, 
          m.name
        FROM memberships m
        LEFT JOIN membershipbenefits mb ON mb.membership_id = m.id
        GROUP BY m.id, m.name
        HAVING COUNT(mb.benefit_id) = 0;
        +----+---------------+
        | id | name          |
        +----+---------------+
        |  3 | Membresía VIP |
        +----+---------------+

11. Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.

        SELECT
          p.id,
          p.name,
          ROUND(AVG(qp.rating), 2) AS avg_rating,
          COUNT(qp.rating) AS num_ratings
        FROM products p
        LEFT JOIN quality_products qp ON qp.product_id = p.id
        WHERE p.category_id = 3 
        GROUP BY p.id, p.name
        ORDER BY avg_rating DESC;
        +-----+--------------+------------+-------------+
        | id  | name         | avg_rating | num_ratings |
        +-----+--------------+------------+-------------+
        | 113 | Producto 113 |       NULL |           0 |
        | 116 | Producto 116 |       NULL |           0 |
        | 119 | Producto 119 |       NULL |           0 |
        | 122 | Producto 122 |       NULL |           0 |
        | 125 | Producto 125 |       NULL |           0 |
        | 128 | Producto 128 |       NULL |           0 |
        +-----+--------------+------------+-------------+

12. Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.

    ```
    SELECT
      f.customer_id,
      COUNT(DISTINCT f.company_id) AS num_companies
    FROM favorites f
    WHERE f.customer_id IS NOT NULL AND f.company_id IS NOT NULL
    GROUP BY f.customer_id
    HAVING COUNT(DISTINCT f.company_id) > 1
    ORDER BY num_companies DESC;
    +-------------+---------------+
    | customer_id | num_companies |
    +-------------+---------------+
    |         105 |             2 |
    |         107 |             2 |
    |         108 |             2 |
    |         109 |             2 |
    |         111 |             2 |
    |         112 |             2 |
    |         113 |             2 |
    |         114 |             2 |
    |         115 |             2 |
    +-------------+---------------+
    ```

13. Como director, quiero identificar las ciudades con más clientes activos.

        SELECT 
              cm.name AS city_name,
              COUNT(c.id) AS total_customers
            FROM customers c
            JOIN citiesormunicipalities cm ON cm.code = c.city_id
            GROUP BY cm.name
            ORDER BY total_customers DESC;
        +------------------+-----------------+
        | city_name        | total_customers |
        +------------------+-----------------+
        | Barranquilla     |               2 |
        | Baranoa          |               2 |
        | Campo de la Cruz |               2 |
        | Candelaria       |               2 |
        | Galapa           |               2 |
        +------------------+-----------------+

14. Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de `quality_products`.

        SELECT
              cp.company_id,
              p.id AS product_id,
              p.name AS product_name,
              AVG(qp.rating) AS avg_rating,
              COUNT(qp.rating) AS num_ratings
            FROM companyproducts cp
            JOIN products p ON p.id = cp.product_id
            LEFT JOIN quality_products qp ON qp.product_id = p.id
            GROUP BY cp.company_id, p.id, p.name
            HAVING COUNT(qp.rating) > 0
            ORDER BY cp.company_id, avg_rating DESC;
        +------------+------------+-----------------------+------------+-------------+
        | company_id | product_id | product_name          | avg_rating | num_ratings |
        +------------+------------+-----------------------+------------+-------------+
        | 900000001  |          7 | Tablet Z10            |   4.850000 |           2 |
        | 900000001  |          3 | Smartphone Alpha      |   4.450000 |           2 |
        | 900000001  |          6 | Auriculares Bluetooth |   4.100000 |           2 |
        | 900000001  |          4 | Monitor 24"           |   4.100000 |           2 |
        | 900000001  |          5 | Teclado Mecánico      |   3.850000 |           2 |
        | 900654321  |          1 | Laptop XYZ            |   4.400000 |           2 |
        | 900654321  |         10 | Router WiFi           |   4.250000 |           2 |
        | 900654321  |          9 | Cámara Web HD         |   4.150000 |           2 |
        | 900654321  |          8 | Impresora Láser       |   4.150000 |           2 |
        +------------+------------+-----------------------+------------+-------------+

15. Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.

        SELECT
          cp.company_id,
          COUNT(DISTINCT cp.product_id) AS total_products
        FROM companyproducts cp
        GROUP BY cp.company_id
        HAVING COUNT(DISTINCT cp.product_id) >= 5 
        ORDER BY total_products DESC;
        +------------+----------------+
        | company_id | total_products |
        +------------+----------------+
        | 900000001  |              5 |
        | 900654321  |              5 |
        | 900987654  |              5 |
        +------------+----------------+

16. Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.

        SELECT 
              p.id, 
              p.name
            FROM products p
            JOIN details_favorites df ON df.product_id = p.id
            LEFT JOIN quality_products qp ON qp.product_id = p.id
            WHERE qp.product_id IS NULL;
        +-----+------------------------+
        | id  | name                   |
        +-----+------------------------+
        | 999 | Producto sin calificar |
        +-----+------------------------+

17. Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.

        SELECT
              a.id AS audience_id,
              a.description AS audience_description,
              b.id AS benefit_id,
              b.description AS benefit_description
            FROM audiencebenefits ab
            JOIN audiences a ON a.id = ab.audience_id
            JOIN benefits b ON b.id = ab.benefit_id
            ORDER BY a.id, b.id;
        +-------------+----------------------+------------+------------------------------+
        | audience_id | audience_description | benefit_id | benefit_description          |
        +-------------+----------------------+------------+------------------------------+
        |           1 | General              |        101 | Acceso a contenido exclusivo |
        |           2 | Estudiantes          |        101 | Acceso a contenido exclusivo |
        |           2 | Estudiantes          |        102 | Descuento en eventos         |
        |           3 | Profesionales        |        103 | Soporte prioritario          |
        |           4 | Adultos Mayores      |        104 | Invitaciones a webinars      |
        |           5 | Empresas             |        102 | Descuento en eventos         |
        |           5 | Empresas             |        103 | Soporte prioritario          |
        +-------------+----------------------+------------+------------------------------+

18. Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.

        SELECT DISTINCT
                ci.code AS city_code,
                ci.name AS city_name
            FROM companies co
            JOIN citiesormunicipalities ci ON ci.code = co.city_id
            LEFT JOIN companyproducts cp ON cp.company_id = co.id
            WHERE cp.product_id IS NULL;
        +-----------+--------------+
        | city_code | city_name    |
        +-----------+--------------+
        | 08001     | Barranquilla |
        +-----------+--------------+

19. Como técnico, deseo obtener todas las empresas con productos duplicados por nombre.

        SELECT
                cp.company_id,
                p.name AS product_name,
                COUNT(*) AS duplicate_count
            FROM companyproducts cp
            JOIN products p ON p.id = cp.product_id
            GROUP BY cp.company_id, p.name
            HAVING COUNT(*) > 1
            ORDER BY cp.company_id, duplicate_count DESC;
        +------------+--------------+-----------------+
        | company_id | product_name | duplicate_count |
        +------------+--------------+-----------------+
        | C001       | Producto X   |               2 |
        +------------+--------------+-----------------+

20. Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido.

        SELECT
                c.id AS customer_id,
                c.name AS customer_name,
                p.id AS product_id,
                p.name AS product_name,
                AVG(qp.rating) AS avg_rating
            FROM customers c
            JOIN favorites f ON f.customer_id = c.id
            JOIN details_favorites df ON df.favorite_id = f.id
            JOIN products p ON p.id = df.product_id
            LEFT JOIN quality_products qp ON qp.product_id = p.id
            GROUP BY c.id, c.name, p.id, p.name
            ORDER BY c.id, p.id;
        +-------------+-----------------+------------+--------------+------------+
        | customer_id | customer_name   | product_id | product_name | avg_rating |
        +-------------+-----------------+------------+--------------+------------+
        |           1 | Carlos Pérez    |          1 | Laptop XYZ   |   4.400000 |
        |           1 | Carlos Pérez    |          2 | Mouse Pro    |   4.550000 |
        |           2 | Ana Gómez       |          1 | Laptop XYZ   |   4.400000 |
        |           2 | Ana Gómez       |          2 | Mouse Pro    |   4.550000 |
        |           3 | Luis Torres     |          1 | Laptop XYZ   |   4.400000 |
        |           3 | Luis Torres     |          2 | Mouse Pro    |   4.550000 |
        |           4 | María Rivas     |          1 | Laptop XYZ   |   4.400000 |
        |           4 | María Rivas     |          2 | Mouse Pro    |   4.550000 |
        |           5 | Diego Mendoza   |          1 | Laptop XYZ   |   4.400000 |
        |           5 | Diego Mendoza   |          2 | Mouse Pro    |   4.550000 |
        |           6 | Sofía Castillo  |          1 | Laptop XYZ   |   4.400000 |
        |           7 | Andrés Molina   |          1 | Laptop XYZ   |   4.400000 |
        |           8 | Luisa Fernández |          1 | Laptop XYZ   |   4.400000 |
        |           9 | Pedro López     |          1 | Laptop XYZ   |   4.400000 |
        |          10 | Carolina Reyes  |          1 | Laptop XYZ   |   4.400000 |
        +-------------+-----------------+------------+--------------+------------+ 

