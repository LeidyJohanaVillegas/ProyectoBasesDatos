## 🔹 **4. Procedimientos Almacenados**

### **1. Registrar una nueva calificación y actualizar el promedio**

> *"Como desarrollador, quiero un procedimiento que registre una calificación y actualice el promedio del producto."*

🧠 **Explicación:** Este procedimiento recibe `product_id`, `customer_id` y `rating`, inserta la nueva fila en `rates`, y recalcula automáticamente el promedio en la tabla `products` (campo `average_rating`).

```
DELIMITER //
CREATE PROCEDURE RegisterAndUpdateRating(
    IN p_product_id INT,
    IN p_customer_id INT,
    IN p_rating DOUBLE(10,2)
)
BEGIN
    INSERT INTO rates (customer_id, company_id, poll_id, daterating, rating)
    SELECT p_customer_id, cp.company_id, 1, NOW(), p_rating
    FROM companyproducts cp
    WHERE cp.product_id = p_product_id
    LIMIT 1
    ON DUPLICATE KEY UPDATE
        daterating = NOW(),
        rating = p_rating;
    UPDATE products p
    SET p.average_rating = (
        SELECT AVG(r.rating)
        FROM rates r
        JOIN companyproducts cp ON r.company_id = cp.company_id
        WHERE cp.product_id = p_product_id
    )
    WHERE p.id = p_product_id;
END //
DELIMITER ;
CALL RegisterAndUpdateRating(24, 1, 4.5);
```

------

### **2. Insertar empresa y asociar productos por defecto**

> *"Como administrador, deseo un procedimiento para insertar una empresa y asociar productos por defecto."*

🧠 **Explicación:** Este procedimiento inserta una empresa en `companies`, y luego vincula automáticamente productos predeterminados en `companyproducts`.

```
DELIMITER //
CREATE PROCEDURE InsertCompanyAndProducts(
    IN p_company_id VARCHAR(20),
    IN p_type_id INT,
    IN p_name VARCHAR(80),
    IN p_category_id INT,
    IN p_city_id VARCHAR(6),
    IN p_audience_id INT,
    IN p_cellphone VARCHAR(15),
    IN p_email VARCHAR(80)
)
BEGIN
    -- 1. Verificar si la ciudad existe en la tabla 'citiesormunicipalities'
    IF NOT EXISTS (SELECT 1 FROM citiesormunicipalities WHERE code = p_city_id) THEN
        -- Si no existe, insertar la ciudad con un nombre genérico
        INSERT INTO citiesormunicipalities (code, name)
        VALUES (p_city_id, CONCAT('Ciudad ', p_city_id));
    END IF;
    -- 2. Insertar la nueva empresa en la tabla 'companies'
    INSERT INTO companies (id, type_id, name, category_id, city_id, audience_id, cellphone, email)
    VALUES (p_company_id, p_type_id, p_name, p_category_id, p_city_id, p_audience_id, p_cellphone, p_email)
    ON DUPLICATE KEY UPDATE
        type_id = p_type_id,
        name = p_name,
        category_id = p_category_id,
        city_id = p_city_id,
        audience_id = p_audience_id,
        cellphone = p_cellphone,
        email = p_email;
    -- 3. Asociar productos a la nueva empresa
    INSERT INTO companyproducts (company_id, product_id, price, unitmeasure_id)
    SELECT p_company_id, p.id, p.price,
           CASE
               WHEN p.category_id = 1 THEN 1
               WHEN p.category_id = 2 THEN 2
               WHEN p.category_id = 3 THEN 6
               WHEN p.category_id = 4 THEN 2
               WHEN p.category_id = 5 THEN 6
               WHEN p.category_id = 6 THEN 5
               WHEN p.category_id = 7 THEN 5
               WHEN p.category_id = 8 THEN 1
               WHEN p.category_id = 9 THEN 1
               WHEN p.category_id = 10 THEN 1
               ELSE 1
           END as unitmeasure_id
    FROM products p
    WHERE p.category_id = p_category_id
    LIMIT 3;
    -- 4. Retornar un mensaje con el resultado
    SELECT CONCAT('Empresa ', p_name, ' creada exitosamente con ID: ', p_company_id) AS mensaje;
END //
DELIMITER ;
CALL InsertCompanyAndProducts('COMP016', 2, 'Nueva Tech Solutions', 1, '05001', 1, '5743001234569', 'info@nuevatech.com');
```

------

### **3. Añadir producto favorito validando duplicados**

> *"Como cliente, quiero un procedimiento que añada un producto favorito y verifique duplicados."*

🧠 **Explicación:** Verifica si el producto ya está en favoritos (`details_favorites`). Si no lo está, lo inserta. Evita duplicaciones silenciosamente.

```
DELIMITER //
CREATE PROCEDURE AddProductToFavorites(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_category_id INT
)
BEGIN
    -- 1. Verificar si el producto ya está en favoritos para este cliente
    IF NOT EXISTS (
        SELECT 1 
        FROM details_favorites 
        WHERE product_id = p_product_id AND category_id = p_category_id
    ) THEN
        -- 2. Si el producto no está en favoritos, insertarlo
        INSERT INTO details_favorites (product_id, category_id)
        VALUES (p_product_id, p_category_id);    
        -- 3. Mostrar mensaje de éxito
        SELECT CONCAT('Producto ', p_product_id, ' añadido a los favoritos de cliente ', p_customer_id) AS mensaje;
    ELSE
        -- 4. Si ya está en favoritos, no hacer nada (evitar duplicados)
        SELECT 'El producto ya está en tus favoritos.' AS mensaje;
    END IF;
END //
DELIMITER ;
CALL AddProductToFavorites(1, 24, 1);
```

------

### **4. Generar resumen mensual de calificaciones por empresa**

> *"Como gestor, deseo un procedimiento que genere un resumen mensual de calificaciones por empresa."*

🧠 **Explicación:** Hace una consulta agregada con `AVG(rating)` por empresa, y guarda los resultados en una tabla de resumen tipo `resumen_calificaciones`.

```
DELIMITER //
CREATE PROCEDURE GenerateMonthlyRatingSummary()
BEGIN       
    -- 1. Crear tabla temporal para almacenar el resumen
    CREATE TEMPORARY TABLE IF NOT EXISTS resumen_calificaciones (
        company_id VARCHAR(20),
        average_rating DOUBLE(10,2),
        total_ratings INT,
        month INT,
        year INT
    );
    -- 2. Insertar datos agregados en la tabla temporal
    INSERT INTO resumen_calificaciones (company_id, average_rating, total_ratings, month, year)
    SELECT 
        cp.company_id,
        AVG(r.rating) AS average_rating,
        COUNT(r.rating) AS total_ratings,
        MONTH(NOW()) AS month,
        YEAR(NOW()) AS year
    FROM 
        rates r
    JOIN 
        companyproducts cp ON r.company_id = cp.company_id
    GROUP BY 
        cp.company_id;
    -- 3. Mostrar el resumen generado
    SELECT * FROM resumen_calificaciones;
END //  
DELIMITER ;
CALL GenerateMonthlyRatingSummary();
```

------

### **5. Calcular beneficios activos por membresía**

> *"Como supervisor, quiero un procedimiento que calcule beneficios activos por membresía."*

🧠 **Explicación:** Consulta `membershipbenefits` junto con `membershipperiods`, y devuelve una lista de beneficios vigentes según la fecha actual.

```
DELIMITER //
CREATE PROCEDURE CalculateActiveMembershipBenefits()    
BEGIN
    -- 1. Seleccionar beneficios activos
    SELECT 
        mb.code AS benefit_id,
        mb.description AS benefit_description,
        mp.start_date,
        mp.end_date
    FROM 
        membershipbenefits mb
    JOIN 
        membershipperiods mp ON mb.membership_id = mp.membership_id
    WHERE 
        NOW() BETWEEN mp.start_date AND mp.end_date;
END //
DELIMITER ;
CALL CalculateActiveMembershipBenefits();
```

------

### **6. Eliminar productos huérfanos**

> *"Como técnico, deseo un procedimiento que elimine productos sin calificación ni empresa asociada."*

🧠 **Explicación:** Elimina productos de la tabla `products` que no tienen relación ni en `rates` ni en `companyproducts`.

```
DELIMITER //
CREATE PROCEDURE DeleteOrphanedProducts()       
BEGIN
    -- 1. Eliminar productos que no están relacionados en rates ni companyproducts
    DELETE FROM products
    WHERE id NOT IN (
        SELECT DISTINCT product_id FROM companyproducts
    ) AND id NOT IN (
        SELECT DISTINCT product_id FROM rates
    );
    -- 2. Mostrar mensaje de éxito
    SELECT 'Productos huérfanos eliminados exitosamente.' AS mensaje;
END //
DELIMITER ;
CALL DeleteOrphanedProducts();

```

------

### **7. Actualizar precios de productos por categoría**

> *"Como operador, quiero un procedimiento que actualice precios de productos por categoría."*

🧠 **Explicación:** Recibe un `categoria_id` y un `factor` (por ejemplo 1.05), y multiplica todos los precios por ese factor en la tabla `companyproducts`.

```
DELIMITER //
CREATE PROCEDURE UpdateProductPricesByCategory( 
    IN p_category_id INT,
    IN p_factor DECIMAL(5,2)
)   
BEGIN
    -- 1. Actualizar precios de productos en companyproducts por categoría
    UPDATE companyproducts cp
    JOIN products p ON cp.product_id = p.id
    SET cp.price = cp.price * p_factor
    WHERE p.category_id = p_category_id;
    -- 2. Mostrar mensaje de éxito
    SELECT CONCAT('Precios actualizados para la categoría ', p_category_id, ' con factor ', p_factor) AS mensaje;
END //
DELIMITER ;
CALL UpdateProductPricesByCategory(1, 1.05);
```

------

### **8. Validar inconsistencia entre `rates` y `quality_products`**

> *"Como auditor, deseo un procedimiento que liste inconsistencias entre `rates` y `quality_products`."*

🧠 **Explicación:** Busca calificaciones (`rates`) que no tengan entrada correspondiente en `quality_products`. Inserta el error en una tabla `errores_log`.

```
DELIMITER //
CREATE PROCEDURE ValidateRatesAndQualityProducts()  
BEGIN
    -- 1. Crear tabla temporal para registrar errores
    CREATE TEMPORARY TABLE IF NOT EXISTS errores_log (
        error_id INT AUTO_INCREMENT PRIMARY KEY,
        error_message VARCHAR(255),
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    -- 2. Insertar errores de inconsistencia
    INSERT INTO errores_log (error_message)
    SELECT 
        CONCAT('Calificación sin producto de calidad: ', r.id)
    FROM 
        rates r
    LEFT JOIN 
        quality_products qp ON r.product_id = qp.product_id
    WHERE 
        qp.product_id IS NULL;
    -- 3. Mostrar los errores registrados
    SELECT * FROM errores_log;
END //
DELIMITER ;
CALL ValidateRatesAndQualityProducts();
```

------

### **9. Asignar beneficios a nuevas audiencias**

> *"Como desarrollador, quiero un procedimiento que asigne beneficios a nuevas audiencias."*

🧠 **Explicación:** Recibe un `benefit_id` y `audience_id`, verifica si ya existe el registro, y si no, lo inserta en `audiencebenefits`.

```
DELIMITER //
CREATE PROCEDURE AssignBenefitsToNewAudience(
    IN p_benefit_id INT,
    IN p_audience_id INT
)   
BEGIN
    -- 1. Verificar si el beneficio ya está asignado a la audiencia
    IF NOT EXISTS (
        SELECT 1 
        FROM audiencebenefits 
        WHERE benefit_id = p_benefit_id AND audience_id = p_audience_id
    ) THEN
        -- 2. Si no existe, insertar el nuevo beneficio para la audiencia
        INSERT INTO audiencebenefits (benefit_id, audience_id)
        VALUES (p_benefit_id, p_audience_id);
        -- 3. Mostrar mensaje de éxito
        SELECT CONCAT('Beneficio ', p_benefit_id, ' asignado a audiencia ', p_audience_id) AS mensaje;
    ELSE
        -- 4. Si ya existe, no hacer nada (evitar duplicados)
        SELECT 'El beneficio ya está asignado a esta audiencia.' AS mensaje;
    END IF;
END //
DELIMITER ;
CALL AssignBenefitsToNewAudience(1, 2);
```

------

### **10. Activar planes de membresía vencidos con pago confirmado**

> *"Como administrador, deseo un procedimiento que active planes de membresía vencidos si el pago fue confirmado."*

🧠 **Explicación:** Actualiza el campo `status` a `'ACTIVA'` en `membershipperiods` donde la fecha haya vencido pero el campo `pago_confirmado` sea `TRUE`.

```
DELIMITER //
CREATE PROCEDURE ActivateExpiredMemberships()   
BEGIN
    -- 1. Actualizar membresías vencidas con pago confirmado
    UPDATE membershipperiods
    SET status = 'ACTIVA'
    WHERE end_date < NOW() AND pago_confirmado = TRUE;
    
    -- 2. Mostrar mensaje de éxito
    SELECT 'Membresías vencidas activadas exitosamente.' AS mensaje;
END //
DELIMITER ;
CALL ActivateExpiredMemberships();
```

------

### **11. Listar productos favoritos del cliente con su calificación**

> *"Como cliente, deseo un procedimiento que me devuelva todos mis productos favoritos con su promedio de rating."*

🧠 **Explicación:** Consulta todos los productos favoritos del cliente y muestra el promedio de calificación de cada uno, uniendo `favorites`, `rates` y `products`.

```
DELIMITER //
CREATE PROCEDURE ListCustomerFavoriteProductsWithRating(
    IN p_customer_id INT
)
BEGIN
    -- 1. Seleccionar productos favoritos con calificación
    SELECT 
        f.product_id,
        p.name AS product_name,
        AVG(r.rating) AS average_rating
    FROM 
        details_favorites f
    JOIN 
        products p ON f.product_id = p.id
    LEFT JOIN 
        rates r ON f.product_id = r.product_id AND r.customer_id = p_customer_id
    WHERE 
        f.customer_id = p_customer_id
    GROUP BY 
        f.product_id, p.name;
END //
DELIMITER ;
CALL ListCustomerFavoriteProductsWithRating(1);
```

------

### **12. Registrar encuesta y sus preguntas asociadas**

> *"Como gestor, quiero un procedimiento que registre una encuesta y sus preguntas asociadas."*

🧠 **Explicación:** Inserta la encuesta principal en `polls` y luego cada una de sus preguntas en otra tabla relacionada como `poll_questions`.

```
DELIMITER //
CREATE PROCEDURE RegisterPollWithQuestions(
    IN p_poll_name VARCHAR(100),
    IN p_poll_description TEXT,
    IN p_questions JSON
)   
BEGIN
    DECLARE v_poll_id INT;
    -- 1. Insertar la encuesta principal
    INSERT INTO polls (name, description)
    VALUES (p_poll_name, p_poll_description);
    -- 2. Obtener el ID de la encuesta recién insertada
    SET v_poll_id = LAST_INSERT_ID();
    -- 3. Insertar las preguntas asociadas desde el JSON
    INSERT INTO poll_questions (poll_id, question_text)
    SELECT v_poll_id, question
    FROM JSON_TABLE(p_questions, '$[*]' COLUMNS (question VARCHAR(255) PATH '$')) AS jt;
    -- 4. Mostrar mensaje de éxito
    SELECT CONCAT('Encuesta "', p_poll_name, '" registrada con Cod: ', v_poll_id) AS mensaje;
END //
DELIMITER ;
CALL RegisterPollWithQuestions('Encuesta de Satisfacción', 'Encuesta para medir la satisfacción del cliente', '["¿Cómo calificaría nuestro servicio?", "¿Qué mejorarías en nuestra plataforma?"]');   
```

------

### **13. Eliminar favoritos antiguos sin calificaciones**

> *"Como técnico, deseo un procedimiento que borre favoritos antiguos no calificados en más de un año."*

🧠 **Explicación:** Filtra productos favoritos que no tienen calificaciones recientes y fueron añadidos hace más de 12 meses, y los elimina de `details_favorites`.

```
DELIMITER //
CREATE PROCEDURE DeleteOldFavoritesWithoutRatings()
BEGIN
    -- 1. Eliminar favoritos antiguos sin calificaciones
    DELETE FROM details_favorites
    WHERE product_id IN (
        SELECT f.product_id
        FROM details_favorites f
        LEFT JOIN rates r ON f.product_id = r.product_id
        WHERE r.rating IS NULL AND f.created_at < NOW() - INTERVAL 12 MONTH
    );
    -- 2. Mostrar mensaje de éxito
    SELECT 'Favoritos antiguos sin calificaciones eliminados exitosamente.' AS mensaje;
END //
DELIMITER ;
CALL DeleteOldFavoritesWithoutRatings();
```

------

### **14. Asociar beneficios automáticamente por audiencia**

> *"Como operador, quiero un procedimiento que asocie automáticamente beneficios por audiencia."*

🧠 **Explicación:** Inserta en `audiencebenefits` todos los beneficios que apliquen según una lógica predeterminada (por ejemplo, por tipo de usuario).

```
DELIMITER //
CREATE PROCEDURE AutoAssignBenefitsByAudience()
BEGIN
    -- 1. Insertar beneficios por audiencia según lógica predeterminada
    INSERT INTO audiencebenefits (benefit_id, audience_id)
    SELECT mb.id, a.id
    FROM membershipbenefits mb
    JOIN audiences a ON mb.audience_id = a.id
    WHERE NOT EXISTS (
        SELECT 1 
        FROM audiencebenefits ab 
        WHERE ab.benefit_id = mb.id AND ab.audience_id = a.id
    );
    -- 2. Mostrar mensaje de éxito
    SELECT 'Beneficios asignados automáticamente por audiencia.' AS mensaje;
END //  
DELIMITER ;
CALL AutoAssignBenefitsByAudience();
```

------

### **15. Historial de cambios de precio**

> *"Como administrador, deseo un procedimiento para generar un historial de cambios de precio."*

🧠 **Explicación:** Cada vez que se cambia un precio, el procedimiento compara el anterior con el nuevo y guarda un registro en una tabla `historial_precios`.

```
DELIMITER //
CREATE PROCEDURE LogPriceChange(
    IN p_product_id INT,
    IN p_old_price DECIMAL(10,2),
    IN p_new_price DECIMAL(10,2)
)   
BEGIN
    -- 1. Insertar el cambio de precio en la tabla de historial
    INSERT INTO historial_precios (product_id, old_price, new_price, change_date)
    VALUES (p_product_id, p_old_price, p_new_price, NOW());
    -- 2. Actualizar el precio del producto en companyproducts
    UPDATE companyproducts
    SET price = p_new_price
    WHERE product_id = p_product_id;
    -- 3. Mostrar mensaje de éxito
    SELECT CONCAT('Cambio de precio registrado para producto ID: ', p_product_id) AS mensaje;
END //
DELIMITER ;
CALL LogPriceChange(24, 100.00, 120.00);
```

------

### **16. Registrar encuesta activa automáticamente**

> *"Como desarrollador, quiero un procedimiento que registre automáticamente una nueva encuesta activa."*

🧠 **Explicación:** Inserta una encuesta en `polls` con el campo `status = 'activa'` y una fecha de inicio en `NOW()`.

```
DELIMITER //
CREATE PROCEDURE RegisterActivePoll(
    IN p_poll_name VARCHAR(100),
    IN p_poll_description TEXT
)
BEGIN
    -- 1. Insertar la encuesta activa
    INSERT INTO polls (name, description, status, start_date)
    VALUES (p_poll_name, p_poll_description, 'activa', NOW());
    -- 2. Mostrar mensaje de éxito
    SELECT CONCAT('Encuesta activa "', p_poll_name, '" registrada exitosamente.') AS mensaje;
END //
DELIMITER ;
CALL RegisterActivePoll('Encuesta de Satisfacción del Cliente', 'Encuesta para medir la satisfacción del cliente con nuestros servicios.');
```

------

### **17. Actualizar unidad de medida de productos sin afectar ventas**

> *"Como técnico, deseo un procedimiento que actualice la unidad de medida de productos sin afectar si hay ventas."*

🧠 **Explicación:** Verifica si el producto no ha sido vendido, y si es así, permite actualizar su `unit_id`.

```
DELIMITER //
CREATE PROCEDURE UpdateProductUnitIfNotSold(
    IN p_product_id INT,
    IN p_new_unit_id INT
)
BEGIN
    -- 1. Verificar si el producto ha sido vendido
    IF NOT EXISTS (
        SELECT 1 
        FROM rates 
        WHERE product_id = p_product_id
    ) THEN
        -- 2. Actualizar la unidad de medida del producto
        UPDATE companyproducts
        SET unitmeasure_id = p_new_unit_id
        WHERE product_id = p_product_id;
        -- 3. Mostrar mensaje de éxito
        SELECT CONCAT('Unidad de medida actualizada para producto ID: ', p_product_id) AS mensaje;
    ELSE
        -- 4. Si el producto ha sido vendido, no permitir actualización
        SELECT 'No se puede actualizar la unidad de medida, el producto ya ha sido vendido.' AS mensaje;
    END IF;
END //  
DELIMITER ;
CALL UpdateProductUnitIfNotSold(24, 2);

```

------

### **18. Recalcular promedios de calidad semanalmente**

> *"Como supervisor, quiero un procedimiento que recalcule todos los promedios de calidad cada semana."*

🧠 **Explicación:** Hace un `AVG(rating)` agrupado por producto y lo actualiza en `products`.

```
DELIMITER //
CREATE PROCEDURE RecalculateWeeklyQualityAverages()
BEGIN
    -- 1. Actualizar el promedio de calificaciones en productos
    UPDATE products p
    SET p.average_rating = (
        SELECT AVG(r.rating)
        FROM rates r
        JOIN companyproducts cp ON r.company_id = cp.company_id
        WHERE cp.product_id = p.id
    );
    -- 2. Mostrar mensaje de éxito
    SELECT 'Promedios de calidad recalculados exitosamente.' AS mensaje;
END //
DELIMITER ;
CALL RecalculateWeeklyQualityAverages();
```

------

### **19. Validar claves foráneas entre calificaciones y encuestas**

> *"Como auditor, deseo un procedimiento que valide claves foráneas cruzadas entre calificaciones y encuestas."*

🧠 **Explicación:** Busca registros en `rates` con `poll_id` que no existen en `polls`, y los reporta.

```
DELIMITER //
CREATE PROCEDURE ValidateForeignKeysBetweenRatesAndPolls()
BEGIN
    -- 1. Seleccionar calificaciones con encuestas inexistentes
    SELECT 
        r.id AS rate_id,
        r.poll_id,
        'Encuesta no encontrada' AS error_message
    FROM 
        rates r
    LEFT JOIN 
        polls p ON r.poll_id = p.id
    WHERE 
        p.id IS NULL;
    -- 2. Mostrar mensaje de validación
    SELECT 'Validación de claves foráneas entre rates y polls completada.' AS mensaje;
END //
DELIMITER ;
CALL ValidateForeignKeysBetweenRatesAndPolls();
```

------

### **20. Generar el top 10 de productos más calificados por ciudad**

> *"Como gerente, quiero un procedimiento que genere el top 10 de productos más calificados por ciudad."*

🧠 **Explicación:** Agrupa las calificaciones por ciudad (a través de la empresa que lo vende) y selecciona los 10 productos con más evaluaciones.

```
DELIMITER //
CREATE PROCEDURE GenerateTop10RatedProductsByCity()
BEGIN
    -- 1. Seleccionar los 10 productos más calificados por ciudad
    SELECT 
        c.name AS city_name,
        p.name AS product_name,
        AVG(r.rating) AS average_rating,
        COUNT(r.rating) AS total_ratings
    FROM 
        rates r
    JOIN 
        companyproducts cp ON r.company_id = cp.company_id
    JOIN 
        products p ON cp.product_id = p.id
    JOIN 
        companies co ON cp.company_id = co.id
    JOIN 
        citiesormunicipalities c ON co.city_id = c.code
    GROUP BY 
        c.name, p.name
    ORDER BY 
        average_rating DESC, total_ratings DESC
    LIMIT 10;
    -- 2. Mostrar mensaje de éxito
    SELECT 'Top 10 productos más calificados por ciudad generado exitosamente.' AS mensaje;
END //
DELIMITER ;
CALL GenerateTop10RatedProductsByCity();
```

