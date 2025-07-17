🔹 **6. Events (Eventos Programados..Usar procedimientos o funciones para cada evento)**
--1. Borrar productos sin actividad cada 6 meses
--Algunos productos pueden haber sido creados pero nunca calificados, marcados como favoritos ni asociados a una empresa. Este evento eliminaría esos productos cada 6 meses.
--Se usaría un DELETE sobre products donde no existan registros en rates, favorites ni companyproducts.
--Frecuencia del evento: EVERY 6 MONTH
DELIMITER //
CREATE PROCEDURE DeleteInactiveProducts()
BEGIN
    DECLARE products_deleted INT DEFAULT 0;
    DELETE FROM products 
    WHERE id NOT IN (
        SELECT DISTINCT product_id FROM rates
        UNION
        SELECT DISTINCT product_id FROM details_favorites
        UNION
        SELECT DISTINCT product_id FROM companyproducts
    );
    SET products_deleted = ROW_COUNT();
    SELECT CONCAT('Cleanup completed: ', products_deleted, ' inactive products deleted') AS result;
END //
DELIMITER ;
CREATE EVENT delete_inactive_products
ON SCHEDULE EVERY 6 MONTH
DO
    CALL DeleteInactiveProducts();

--2. Recalcular el promedio de calificaciones semanalmente
--Se puede tener una tabla product_metrics que almacena promedios pre-calculados para rapidez. El evento actualizaría esa tabla con nuevos promedios.
--Usa UPDATE con AVG(rating) agrupado por producto.
--Frecuencia: EVERY 1 WEEK
DELIMITER //
CREATE EVENT recalculate_product_rating_avg
ON SCHEDULE EVERY 1 WEEK
DO
    UPDATE product_metrics pm
    JOIN (
        SELECT product_id, AVG(rating) AS avg_rating
        FROM rates
        GROUP BY product_id
    ) r ON pm.product_id = r.product_id
    SET pm.average_rating = r.avg_rating;
DELIMITER ;


--3. Actualizar precios según inflación mensual
--Aplicar un porcentaje de aumento (por ejemplo, 3%) a los precios de todos los productos.
--UPDATE companyproducts SET price = price * 1.03;
--Frecuencia: EVERY 1 MONTH
DELIMITER //
CREATE EVENT update_prices_inflation
ON SCHEDULE EVERY 1 MONTH
DO
    UPDATE companyproducts
    SET price = price * 1.03;
DELIMITER ;

--4. Crear backups lógicos diariamente
--Este evento no ejecuta comandos del sistema, pero puede volcar datos clave a una tabla temporal o de respaldo (products_backup, rates_backup, etc.).
--EVERY 1 DAY STARTS '00:00:00'
DELIMITER //
CREATE EVENT backup_data_daily
ON SCHEDULE EVERY 1 DAY STARTS '2025-07-17 00:00:00'
DO
    INSERT INTO products_backup SELECT * FROM products;
    INSERT INTO rates_backup SELECT * FROM rates;
DELIMITER ;

--5. Notificar sobre productos favoritos sin calificar
--Genera una lista (user_reminders) de product_id donde el cliente tiene el producto en favoritos pero no hay rate.
--Requiere INSERT INTO recordatorios usando un LEFT JOIN y WHERE rate IS NULL.
DELIMITER //
CREATE EVENT notify_unrated_favorites
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO user_reminders (customer_id, product_id)
    SELECT f.customer_id, f.product_id
    FROM details_favorites f
    LEFT JOIN rates r ON f.product_id = r.product_id AND f.customer_id = r.customer_id
    WHERE r.rating IS NULL;
DELIMITER ;

--6. Revisar inconsistencias entre empresa y productos
--Detecta productos sin empresa, o empresas sin productos, y los registra en una tabla de anomalías.
--Puede usar NOT EXISTS y JOIN para llenar una tabla errores_log.
--EVERY 1 WEEK ON SUNDAY
DELIMITER //
CREATE EVENT notify_unrated_favorites
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO user_reminders (customer_id, product_id)
    SELECT f.customer_id, f.product_id
    FROM details_favorites f
    LEFT JOIN rates r ON f.product_id = r.product_id AND f.customer_id = r.customer_id
    WHERE r.rating IS NULL;
DELIMITER ;
--7. Archivar membresías vencidas diariamente
--Cambia el estado de la membresía cuando su end_date ya pasó.
--UPDATE membershipperiods SET status = 'INACTIVA' WHERE end_date < CURDATE();
DELIMITER //
CREATE EVENT archive_expired_memberships
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE membershipperiods
    SET status = 'INACTIVA'
    WHERE end_date < CURDATE();
DELIMITER ;

--8. Notificar beneficios nuevos a usuarios semanalmente
--Detecta registros nuevos en la tabla benefits desde la última semana y los inserta en notificaciones.
--INSERT INTO notificaciones SELECT ... WHERE created_at >= NOW() - INTERVAL 7 DAY
DELIMITER //
CREATE EVENT notify_new_benefits
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO notificaciones (cliente_id, mensaje, fecha)
    SELECT DISTINCT u.cliente_id, CONCAT('Nuevo beneficio: ', b.name), NOW()
    FROM benefits b
    JOIN users u ON u.cliente_id = b.created_by
    WHERE b.created_at >= NOW() - INTERVAL 7 DAY;
DELIMITER ;

--9. Calcular cantidad de favoritos por cliente mensualmente
--Cuenta los productos favoritos por cliente y guarda el resultado en una tabla de resumen mensual (favoritos_resumen).
--INSERT INTO favoritos_resumen SELECT customer_id, COUNT(*) ... GROUP BY customer_id
DELIMITER //
CREATE EVENT calculate_favorites_per_customer
ON SCHEDULE EVERY 1 MONTH
DO
    INSERT INTO favoritos_resumen (customer_id, favorite_count)
    SELECT customer_id, COUNT(*) 
    FROM details_favorites
    GROUP BY customer_id;
DELIMITER ;

--10. Validar claves foráneas semanalmente
--Comprueba que cada product_id, customer_id, etc., tengan correspondencia en sus tablas. Si no, se registra en una tabla inconsistencias_fk.
DELIMITER //
CREATE EVENT validate_foreign_keys
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO inconsistencias_fk (error_type, table_name, invalid_id)
    SELECT 'Product FK', 'rates', r.product_id 
    FROM rates r 
    WHERE NOT EXISTS (SELECT 1 FROM products p WHERE p.id = r.product_id)
    UNION
    SELECT 'Customer FK', 'rates', r.customer_id
    FROM rates r
    WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.id = r.customer_id);
DELIMITER ;

--11. Eliminar calificaciones inválidas antiguas
--Borra rates donde el valor de rating es NULL o <0 y que hayan sido creadas hace más de 3 meses.
--DELETE FROM rates WHERE rating IS NULL AND created_at < NOW() - INTERVAL 3 MONTH
DELIMITER //
CREATE EVENT delete_invalid_ratings
ON SCHEDULE EVERY 1 MONTH
DO
    DELETE FROM rates
    WHERE (rating IS NULL OR rating < 0) AND created_at < NOW() - INTERVAL 3 MONTH;
DELIMITER ;

--12. Cambiar estado de encuestas inactivas automáticamente
--Cambia el campo status = 'inactiva' si una encuesta no tiene nuevas respuestas en más de 6 meses.
DELIMITER //
CREATE EVENT deactivate_old_polls
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE polls
    SET status = 'INACTIVA'
    WHERE id NOT IN (SELECT DISTINCT poll_id FROM responses WHERE response_date >= NOW() - INTERVAL 6 MONTH)
    AND status != 'INACTIVA';
DELIMITER ;

--13. Registrar auditorías de forma periódica
--Cada día, se puede registrar el conteo de productos, usuarios, etc. en una tabla tipo auditorias_diarias.
DELIMITER //
CREATE EVENT daily_audit_log
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO auditorias_diarias (productos_count, clientes_count, fecha)
    SELECT 
        (SELECT COUNT(*) FROM products),
        (SELECT COUNT(*) FROM customers),
        NOW();
DELIMITER ;

--14. Notificar métricas de calidad a empresas
--Genera una tabla o archivo con AVG(rating) por producto y empresa y se registra en notificaciones_empresa.
DELIMITER //
CREATE EVENT notify_quality_metrics
ON SCHEDULE EVERY 1 WEEK ON MONDAY
DO
    INSERT INTO notificaciones_empresa (company_id, message)
    SELECT company_id, CONCAT('Promedio de calidad para el producto ', p.id, ' es ', AVG(r.rating))
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    JOIN rates r ON r.product_id = p.id
    GROUP BY company_id;
DELIMITER ;

--15. Recordar renovación de membresías
--Busca membershipperiods donde end_date esté entre hoy y 7 días adelante, e inserta recordatorios.

DELIMITER //
CREATE EVENT remind_membership_renewal
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO recordatorios (customer_id, message, reminder_date)
    SELECT customer_id, CONCAT('Renueva tu membresía, vence el ', end_date), NOW()
    FROM membershipperiods
    WHERE end_date BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY;
DELIMITER ;

--16. Reordenar estadísticas generales cada semana
--Calcula y actualiza métricas como total de productos activos, clientes registrados, etc., en una tabla estadisticas.
DELIMITER //
CREATE EVENT reorder_general_statistics
ON SCHEDULE EVERY 1 WEEK
DO
    UPDATE general_statistics
    SET active_products = (SELECT COUNT(*) FROM products WHERE status = 'ACTIVO'),
        registered_clients = (SELECT COUNT(*) FROM customers)
    WHERE id = 1; -- Suponiendo que hay una fila para las estadísticas
DELIMITER ;

--17. Crear resúmenes temporales de uso por categoría
--Cuenta cuántos productos se han calificado en cada categoría y guarda los resultados para dashboards.
DELIMITER //
CREATE EVENT create_category_usage_summary
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO category_usage_summary (category_id, rated_products_count)
    SELECT p.category_id, COUNT(*)
    FROM products p
    JOIN rates r ON r.product_id = p.id
    GROUP BY p.category_id;
DELIMITER ;

--18. Actualizar beneficios caducados
--Revisa si un beneficio tiene una fecha de expiración (campo expires_at) y lo marca como inactivo.
DELIMITER //
CREATE EVENT deactivate_expired_benefits
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE benefits
    SET status = 'INACTIVO'
    WHERE expires_at < CURDATE() AND status != 'INACTIVO';
DELIMITER ;

--19. Alertar productos sin evaluación anual
--Busca productos sin rate en los últimos 365 días y genera alertas o registros en alertas_productos.
DELIMITER //
CREATE EVENT alert_no_annual_reviews
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO alertas_productos (product_id, alert_message)
    SELECT p.id, 'Producto sin evaluación en el último año'
    FROM products p
    LEFT JOIN rates r ON r.product_id = p.id AND r.created_at >= CURDATE() - INTERVAL 1 YEAR
    WHERE r.product_id IS NULL;
DELIMITER ;

--20. Actualizar precios con índice externo
--Se podría tener una tabla inflacion_indice y aplicar ese valor multiplicador a los precios de productos activos.
DELIMITER //
CREATE EVENT update_prices_with_external_index
ON SCHEDULE EVERY 1 MONTH
DO
    UPDATE companyproducts cp
    JOIN inflacion_indice i ON i.id = 1  
    SET cp.price = cp.price * i.inflation_rate;
DELIMITER ;
--1. Borrar productos sin actividad cada 6 meses
--Algunos productos pueden haber sido creados pero nunca calificados, marcados como favoritos ni asociados a una empresa. Este evento eliminaría esos productos cada 6 meses.
--Se usaría un DELETE sobre products donde no existan registros en rates, favorites ni companyproducts.
--Frecuencia del evento: EVERY 6 MONTH
DELIMITER //
CREATE PROCEDURE DeleteInactiveProducts()
BEGIN
    DECLARE products_deleted INT DEFAULT 0;
    DELETE FROM products 
    WHERE id NOT IN (
        SELECT DISTINCT product_id FROM rates
        UNION
        SELECT DISTINCT product_id FROM details_favorites
        UNION
        SELECT DISTINCT product_id FROM companyproducts
    );
    SET products_deleted = ROW_COUNT();
    SELECT CONCAT('Cleanup completed: ', products_deleted, ' inactive products deleted') AS result;
END //
DELIMITER ;
CREATE EVENT delete_inactive_products
ON SCHEDULE EVERY 6 MONTH
DO
    CALL DeleteInactiveProducts();

--2. Recalcular el promedio de calificaciones semanalmente
--Se puede tener una tabla product_metrics que almacena promedios pre-calculados para rapidez. El evento actualizaría esa tabla con nuevos promedios.
--Usa UPDATE con AVG(rating) agrupado por producto.
--Frecuencia: EVERY 1 WEEK
DELIMITER //
CREATE EVENT recalculate_product_rating_avg
ON SCHEDULE EVERY 1 WEEK
DO
    UPDATE product_metrics pm
    JOIN (
        SELECT product_id, AVG(rating) AS avg_rating
        FROM rates
        GROUP BY product_id
    ) r ON pm.product_id = r.product_id
    SET pm.average_rating = r.avg_rating;
DELIMITER ;


--3. Actualizar precios según inflación mensual
--Aplicar un porcentaje de aumento (por ejemplo, 3%) a los precios de todos los productos.
--UPDATE companyproducts SET price = price * 1.03;
--Frecuencia: EVERY 1 MONTH
DELIMITER //
CREATE EVENT update_prices_inflation
ON SCHEDULE EVERY 1 MONTH
DO
    UPDATE companyproducts
    SET price = price * 1.03;
DELIMITER ;

--4. Crear backups lógicos diariamente
--Este evento no ejecuta comandos del sistema, pero puede volcar datos clave a una tabla temporal o de respaldo (products_backup, rates_backup, etc.).
--EVERY 1 DAY STARTS '00:00:00'
DELIMITER //
CREATE EVENT backup_data_daily
ON SCHEDULE EVERY 1 DAY STARTS '2025-07-17 00:00:00'
DO
    INSERT INTO products_backup SELECT * FROM products;
    INSERT INTO rates_backup SELECT * FROM rates;
DELIMITER ;

--5. Notificar sobre productos favoritos sin calificar
--Genera una lista (user_reminders) de product_id donde el cliente tiene el producto en favoritos pero no hay rate.
--Requiere INSERT INTO recordatorios usando un LEFT JOIN y WHERE rate IS NULL.
DELIMITER //
CREATE EVENT notify_unrated_favorites
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO user_reminders (customer_id, product_id)
    SELECT f.customer_id, f.product_id
    FROM details_favorites f
    LEFT JOIN rates r ON f.product_id = r.product_id AND f.customer_id = r.customer_id
    WHERE r.rating IS NULL;
DELIMITER ;

--6. Revisar inconsistencias entre empresa y productos
--Detecta productos sin empresa, o empresas sin productos, y los registra en una tabla de anomalías.
--Puede usar NOT EXISTS y JOIN para llenar una tabla errores_log.
--EVERY 1 WEEK ON SUNDAY
DELIMITER //
CREATE EVENT notify_unrated_favorites
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO user_reminders (customer_id, product_id)
    SELECT f.customer_id, f.product_id
    FROM details_favorites f
    LEFT JOIN rates r ON f.product_id = r.product_id AND f.customer_id = r.customer_id
    WHERE r.rating IS NULL;
DELIMITER ;
--7. Archivar membresías vencidas diariamente
--Cambia el estado de la membresía cuando su end_date ya pasó.
--UPDATE membershipperiods SET status = 'INACTIVA' WHERE end_date < CURDATE();
DELIMITER //
CREATE EVENT archive_expired_memberships
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE membershipperiods
    SET status = 'INACTIVA'
    WHERE end_date < CURDATE();
DELIMITER ;

--8. Notificar beneficios nuevos a usuarios semanalmente
--Detecta registros nuevos en la tabla benefits desde la última semana y los inserta en notificaciones.
--INSERT INTO notificaciones SELECT ... WHERE created_at >= NOW() - INTERVAL 7 DAY
DELIMITER //
CREATE EVENT notify_new_benefits
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO notificaciones (cliente_id, mensaje, fecha)
    SELECT DISTINCT u.cliente_id, CONCAT('Nuevo beneficio: ', b.name), NOW()
    FROM benefits b
    JOIN users u ON u.cliente_id = b.created_by
    WHERE b.created_at >= NOW() - INTERVAL 7 DAY;
DELIMITER ;

--9. Calcular cantidad de favoritos por cliente mensualmente
--Cuenta los productos favoritos por cliente y guarda el resultado en una tabla de resumen mensual (favoritos_resumen).
--INSERT INTO favoritos_resumen SELECT customer_id, COUNT(*) ... GROUP BY customer_id
DELIMITER //
CREATE EVENT calculate_favorites_per_customer
ON SCHEDULE EVERY 1 MONTH
DO
    INSERT INTO favoritos_resumen (customer_id, favorite_count)
    SELECT customer_id, COUNT(*) 
    FROM details_favorites
    GROUP BY customer_id;
DELIMITER ;

--10. Validar claves foráneas semanalmente
--Comprueba que cada product_id, customer_id, etc., tengan correspondencia en sus tablas. Si no, se registra en una tabla inconsistencias_fk.
DELIMITER //
CREATE EVENT validate_foreign_keys
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO inconsistencias_fk (error_type, table_name, invalid_id)
    SELECT 'Product FK', 'rates', r.product_id 
    FROM rates r 
    WHERE NOT EXISTS (SELECT 1 FROM products p WHERE p.id = r.product_id)
    UNION
    SELECT 'Customer FK', 'rates', r.customer_id
    FROM rates r
    WHERE NOT EXISTS (SELECT 1 FROM customers c WHERE c.id = r.customer_id);
DELIMITER ;

--11. Eliminar calificaciones inválidas antiguas
--Borra rates donde el valor de rating es NULL o <0 y que hayan sido creadas hace más de 3 meses.
--DELETE FROM rates WHERE rating IS NULL AND created_at < NOW() - INTERVAL 3 MONTH
DELIMITER //
CREATE EVENT delete_invalid_ratings
ON SCHEDULE EVERY 1 MONTH
DO
    DELETE FROM rates
    WHERE (rating IS NULL OR rating < 0) AND created_at < NOW() - INTERVAL 3 MONTH;
DELIMITER ;

--12. Cambiar estado de encuestas inactivas automáticamente
--Cambia el campo status = 'inactiva' si una encuesta no tiene nuevas respuestas en más de 6 meses.
DELIMITER //
CREATE EVENT deactivate_old_polls
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE polls
    SET status = 'INACTIVA'
    WHERE id NOT IN (SELECT DISTINCT poll_id FROM responses WHERE response_date >= NOW() - INTERVAL 6 MONTH)
    AND status != 'INACTIVA';
DELIMITER ;

--13. Registrar auditorías de forma periódica
--Cada día, se puede registrar el conteo de productos, usuarios, etc. en una tabla tipo auditorias_diarias.
DELIMITER //
CREATE EVENT daily_audit_log
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO auditorias_diarias (productos_count, clientes_count, fecha)
    SELECT 
        (SELECT COUNT(*) FROM products),
        (SELECT COUNT(*) FROM customers),
        NOW();
DELIMITER ;

--14. Notificar métricas de calidad a empresas
--Genera una tabla o archivo con AVG(rating) por producto y empresa y se registra en notificaciones_empresa.
DELIMITER //
CREATE EVENT notify_quality_metrics
ON SCHEDULE EVERY 1 WEEK ON MONDAY
DO
    INSERT INTO notificaciones_empresa (company_id, message)
    SELECT company_id, CONCAT('Promedio de calidad para el producto ', p.id, ' es ', AVG(r.rating))
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    JOIN rates r ON r.product_id = p.id
    GROUP BY company_id;
DELIMITER ;

--15. Recordar renovación de membresías
--Busca membershipperiods donde end_date esté entre hoy y 7 días adelante, e inserta recordatorios.

DELIMITER //
CREATE EVENT remind_membership_renewal
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO recordatorios (customer_id, message, reminder_date)
    SELECT customer_id, CONCAT('Renueva tu membresía, vence el ', end_date), NOW()
    FROM membershipperiods
    WHERE end_date BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY;
DELIMITER ;

--16. Reordenar estadísticas generales cada semana
--Calcula y actualiza métricas como total de productos activos, clientes registrados, etc., en una tabla estadisticas.
DELIMITER //
CREATE EVENT reorder_general_statistics
ON SCHEDULE EVERY 1 WEEK
DO
    UPDATE general_statistics
    SET active_products = (SELECT COUNT(*) FROM products WHERE status = 'ACTIVO'),
        registered_clients = (SELECT COUNT(*) FROM customers)
    WHERE id = 1; -- Suponiendo que hay una fila para las estadísticas
DELIMITER ;

--17. Crear resúmenes temporales de uso por categoría
--Cuenta cuántos productos se han calificado en cada categoría y guarda los resultados para dashboards.
DELIMITER //
CREATE EVENT create_category_usage_summary
ON SCHEDULE EVERY 1 WEEK
DO
    INSERT INTO category_usage_summary (category_id, rated_products_count)
    SELECT p.category_id, COUNT(*)
    FROM products p
    JOIN rates r ON r.product_id = p.id
    GROUP BY p.category_id;
DELIMITER ;

--18. Actualizar beneficios caducados
--Revisa si un beneficio tiene una fecha de expiración (campo expires_at) y lo marca como inactivo.
DELIMITER //
CREATE EVENT deactivate_expired_benefits
ON SCHEDULE EVERY 1 DAY
DO
    UPDATE benefits
    SET status = 'INACTIVO'
    WHERE expires_at < CURDATE() AND status != 'INACTIVO';
DELIMITER ;

--19. Alertar productos sin evaluación anual
--Busca productos sin rate en los últimos 365 días y genera alertas o registros en alertas_productos.
DELIMITER //
CREATE EVENT alert_no_annual_reviews
ON SCHEDULE EVERY 1 DAY
DO
    INSERT INTO alertas_productos (product_id, alert_message)
    SELECT p.id, 'Producto sin evaluación en el último año'
    FROM products p
    LEFT JOIN rates r ON r.product_id = p.id AND r.created_at >= CURDATE() - INTERVAL 1 YEAR
    WHERE r.product_id IS NULL;
DELIMITER ;

--20. Actualizar precios con índice externo
--Se podría tener una tabla inflacion_indice y aplicar ese valor multiplicador a los precios de productos activos.
DELIMITER //
CREATE EVENT update_prices_with_external_index
ON SCHEDULE EVERY 1 MONTH
DO
    UPDATE companyproducts cp
    JOIN inflacion_indice i ON i.id = 1  
    SET cp.price = cp.price * i.inflation_rate;
DELIMITER ;
