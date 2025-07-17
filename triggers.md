## 🔹 **5. Triggers**

1. ### 🔎 **1. Actualizar la fecha de modificación de un producto**

   > "Como desarrollador, deseo un trigger que actualice la fecha de modificación cuando se actualice un producto."

   🧠 **Explicación:** Cada vez que se actualiza un producto, queremos que el campo `updated_at` se actualice automáticamente con la fecha actual (`NOW()`), sin tener que hacerlo manualmente desde la app.

   🔁 Se usa un `BEFORE UPDATE`.

   ```
   DELIMITER //
   CREATE TRIGGER update_product_updated_at
   BEFORE UPDATE ON products
   FOR EACH ROW
   BEGIN
       SET NEW.updated_at = NOW();
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **2. Registrar log cuando un cliente califica un producto**

   > "Como administrador, quiero un trigger que registre en log cuando un cliente califica un producto."

   🧠 **Explicación:** Cuando alguien inserta una fila en `rates`, el trigger crea automáticamente un registro en `log_acciones` con la información del cliente y producto calificado.

   🔁 Se usa un `AFTER INSERT` sobre `rates`.

   ```
   DELIMITER //
   CREATE TRIGGER log_rate_insert  
   AFTER INSERT ON rates
   FOR EACH ROW
   BEGIN
       INSERT INTO log_acciones (accion, cliente_id, producto_id, fecha)
       VALUES ('Calificación de producto', NEW.cliente_id, NEW.producto_id, NOW());
   END //  
   DELIMITER ;
   ```

   ------

   ### 🔎 **3. Impedir insertar productos sin unidad de medida**

   > "Como técnico, deseo un trigger que impida insertar productos sin unidad de medida."

   🧠 **Explicación:** Antes de guardar un nuevo producto, el trigger revisa si `unit_id` es `NULL`. Si lo es, lanza un error con `SIGNAL`.

   🔁 Se usa un `BEFORE INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_null_unit_id
   BEFORE INSERT ON products
   FOR EACH ROW
   BEGIN
       IF NEW.unit_id IS NULL THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El producto debe tener una unidad de medida asignada.';
       END IF;
   END //  
   DELIMITER ;
   ```

   ------

   ### 🔎 **4. Validar calificaciones no mayores a 5**

   > "Como auditor, quiero un trigger que verifique que las calificaciones no superen el valor máximo permitido."

   🧠 **Explicación:** Si alguien intenta insertar una calificación de 6 o más, se bloquea automáticamente. Esto evita errores o trampa.

   🔁 Se usa un `BEFORE INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_null_unit_id
   BEFORE INSERT ON products
   FOR EACH ROW
   BEGIN
       IF NEW.unit_id IS NULL THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El producto debe tener una unidad de medida asignada.';
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **5. Actualizar estado de membresía cuando vence**

   > "Como supervisor, deseo un trigger que actualice automáticamente el estado de membresía al vencer el periodo."

   🧠 **Explicación:** Cuando se actualiza un periodo de membresía (`membershipperiods`), si `end_date` ya pasó, se puede cambiar el campo `status` a 'INACTIVA'.

   🔁 `AFTER UPDATE` o `BEFORE UPDATE` dependiendo de la lógica.

   ```
   DELIMITER //
   CREATE TRIGGER update_membership_status
   AFTER UPDATE ON membershipperiods
   FOR EACH ROW
   BEGIN
       IF NEW.end_date < NOW() THEN
           UPDATE membershipperiods
           SET status = 'INACTIVA'
           WHERE membership_id = NEW.membership_id
           AND period_id = NEW.period_id;
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **6. Evitar duplicados de productos por empresa**

   > "Como operador, quiero un trigger que evite duplicar productos por nombre dentro de una misma empresa."

   🧠 **Explicación:** Antes de insertar un nuevo producto en `companyproducts`, el trigger puede consultar si ya existe uno con el mismo `product_id` y `company_id`.

   🔁 `BEFORE INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_duplicate_company_product    
   BEFORE INSERT ON companyproducts
   FOR EACH ROW
   BEGIN
       DECLARE duplicate_count INT;
       SELECT COUNT(*) INTO duplicate_count
       FROM companyproducts
       WHERE product_id = NEW.product_id AND company_id = NEW.company_id;
       IF duplicate_count > 0 THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El producto ya existe para esta empresa.';
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **7. Enviar notificación al añadir un favorito**

   > "Como cliente, deseo un trigger que envíe notificación cuando añado un producto como favorito."

   🧠 **Explicación:** Después de un `INSERT` en `details_favorites`, el trigger agrega un mensaje a una tabla `notificaciones`.

   🔁 `AFTER INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER notify_favorite_added
   AFTER INSERT ON details_favorites
   FOR EACH ROW
   BEGIN
       INSERT INTO notificaciones (cliente_id, mensaje, fecha)
       VALUES (NEW.cliente_id, CONCAT('Producto ', NEW.product_id, ' añadido a favoritos.'), NOW());
   END //  
   DELIMITER ;
   ```

   ------

   ### 🔎 **8. Insertar fila en `quality_products` tras calificación**

   > "Como técnico, quiero un trigger que inserte una fila en `quality_products` cuando se registra una calificación."

   🧠 **Explicación:** Al insertar una nueva calificación en `rates`, se crea automáticamente un registro en `quality_products` para mantener métricas de calidad.

   🔁 `AFTER INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER insert_quality_product
   AFTER INSERT ON rates
   FOR EACH ROW
   BEGIN
       INSERT INTO quality_products (product_id, rate, created_at)
       VALUES (NEW.product_id, NEW.rate, NOW());
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **9. Eliminar favoritos si se elimina el producto**

   > "Como desarrollador, deseo un trigger que elimine los favoritos si se elimina el producto."

   🧠 **Explicación:** Cuando se borra un producto, el trigger elimina las filas en `details_favorites` donde estaba ese producto.

   🔁 `AFTER DELETE` en `products`.

   ```
   DELIMITER //
   CREATE TRIGGER delete_favorites_on_product_delete
   AFTER DELETE ON products
   FOR EACH ROW
   BEGIN
       DELETE FROM details_favorites
       WHERE producto_id = OLD.id;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **10. Bloquear modificación de audiencias activas**

   > "Como administrador, quiero un trigger que bloquee la modificación de audiencias activas."

   🧠 **Explicación:** Si un usuario intenta modificar una audiencia que está en uso, el trigger lanza un error con `SIGNAL`.

   🔁 `BEFORE UPDATE`.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_audience_update_if_active
   BEFORE UPDATE ON audiences
   FOR EACH ROW
   BEGIN
       IF OLD.status = 'ACTIVA' THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se puede modificar una audiencia activa.';
       END IF;
   END //  
   DELIMITER ;
   ```

   ------

   ### 🔎 **11. Recalcular promedio de calidad del producto tras nueva evaluación**

   > "Como gestor, deseo un trigger que actualice el promedio de calidad del producto tras una nueva evaluación."

   🧠 **Explicación:** Después de insertar en `rates`, el trigger actualiza el campo `average_rating` del producto usando `AVG()`.

   🔁 `AFTER INSERT`.

   ```
   DELIMITER //
   CREATE TRIGGER update_product_average_rating
   AFTER INSERT ON rates
   FOR EACH ROW
   BEGIN
       UPDATE products
       SET average_rating = (
           SELECT AVG(rate)
           FROM rates
           WHERE product_id = NEW.product_id
       )
       WHERE id = NEW.product_id;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **12. Registrar asignación de nuevo beneficio**

   > "Como auditor, quiero un trigger que registre cada vez que se asigna un nuevo beneficio."

   🧠 **Explicación:** Cuando se hace `INSERT` en `membershipbenefits` o `audiencebenefits`, se agrega un log en `bitacora`.

   ```
   DELIMITER //
   CREATE TRIGGER log_benefit_assignment
   AFTER INSERT ON membershipbenefits
   FOR EACH ROW
   BEGIN
       INSERT INTO bitacora (accion, descripcion, fecha)
       VALUES ('Asignación de beneficio', CONCAT('Beneficio ', NEW.benefit_id, ' asignado a membresía ', NEW.membership_id), NOW());
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **13. Impedir doble calificación por parte del cliente**

   > "Como cliente, deseo un trigger que me impida calificar el mismo producto dos veces seguidas."

   🧠 **Explicación:** Antes de insertar en `rates`, el trigger verifica si ya existe una calificación de ese `customer_id` y `product_id`.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_duplicate_rate
   BEFORE INSERT ON rates
   FOR EACH ROW
   BEGIN
       DECLARE existing_rate_count INT;
       SELECT COUNT(*) INTO existing_rate_count
       FROM rates
       WHERE customer_id = NEW.customer_id AND product_id = NEW.product_id;
       
       IF existing_rate_count > 0 THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El cliente ya ha calificado este producto.';
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **14. Validar correos duplicados en clientes**

   > "Como técnico, quiero un trigger que valide que el email del cliente no se repita."

   🧠 **Explicación:** Verifica, antes del `INSERT`, si el correo ya existe en la tabla `customers`. Si sí, lanza un error.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_duplicate_email
   BEFORE INSERT ON customers
   FOR EACH ROW
   BEGIN
       DECLARE email_count INT;
       SELECT COUNT(*) INTO email_count
       FROM customers
       WHERE email = NEW.email;
       
       IF email_count > 0 THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El correo electrónico ya está registrado.';
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **15. Eliminar detalles de favoritos huérfanos**

   > "Como operador, deseo un trigger que elimine registros huérfanos de `details_favorites`."

   🧠 **Explicación:** Si se elimina un registro de `favorites`, se borran automáticamente sus detalles asociados.

   ```
   DELIMITER //
   CREATE TRIGGER delete_orphaned_favorite_details
   AFTER DELETE ON favorites
   FOR EACH ROW    
   BEGIN
       DELETE FROM details_favorites
       WHERE favorite_id = OLD.id;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **16. Actualizar campo `updated_at` en `companies`**

   > "Como administrador, quiero un trigger que actualice el campo `updated_at` en `companies`."

   🧠 **Explicación:** Como en productos, actualiza automáticamente la fecha de última modificación cada vez que se cambia algún dato.

   ```
   DELIMITER //
   CREATE TRIGGER update_company_updated_at    
   BEFORE UPDATE ON companies
   FOR EACH ROW
   BEGIN
       SET NEW.updated_at = NOW();
   END //  
   DELIMITER ;
   ```

   ------

   ### 🔎 **17. Impedir borrar ciudad si hay empresas activas**

   > "Como desarrollador, deseo un trigger que impida borrar una ciudad si hay empresas activas en ella."

   🧠 **Explicación:** Antes de hacer `DELETE` en `citiesormunicipalities`, el trigger revisa si hay empresas registradas en esa ciudad.

   ```
   DELIMITER //
   CREATE TRIGGER prevent_city_deletion_if_active_companies
   BEFORE DELETE ON citiesormunicipalities
   FOR EACH ROW
   BEGIN
       DECLARE active_company_count INT;
       SELECT COUNT(*) INTO active_company_count
       FROM companies
       WHERE city_id = OLD.id AND status = 'ACTIVA';
       IF active_company_count > 0 THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No se puede eliminar la ciudad porque hay empresas activas asociadas.';
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **18. Registrar cambios de estado en encuestas**

   > "Como auditor, quiero un trigger que registre cambios de estado de encuestas."

   🧠 **Explicación:** Cada vez que se actualiza el campo `status` en `polls`, el trigger guarda la fecha, nuevo estado y usuario en un log.

   ```
   DELIMITER //
   CREATE TRIGGER log_poll_status_change
   AFTER UPDATE ON polls
   FOR EACH ROW
   BEGIN
       IF OLD.status <> NEW.status THEN
           INSERT INTO poll_status_log (poll_id, old_status, new_status, changed_at, user_id)
           VALUES (NEW.id, OLD.status, NEW.status, NOW(), NEW.updated_by);
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **19. Sincronizar `rates` y `quality_products`**

   > "Como supervisor, deseo un trigger que sincronice `rates` con `quality_products` al calificar."

   🧠 **Explicación:** Inserta o actualiza la calidad del producto en `quality_products` cada vez que se inserta una nueva calificación.

   ```
   DELIMITER //
   CREATE TRIGGER sync_quality_product_after_rate_insert
   AFTER INSERT ON rates
   FOR EACH ROW
   BEGIN
       DECLARE existing_quality_product_count INT;
       SELECT COUNT(*) INTO existing_quality_product_count
       FROM quality_products
       WHERE product_id = NEW.product_id;
       IF existing_quality_product_count > 0 THEN
           UPDATE quality_products
           SET rate = NEW.rate, updated_at = NOW()
           WHERE product_id = NEW.product_id;
       ELSE
           INSERT INTO quality_products (product_id, rate, created_at)
           VALUES (NEW.product_id, NEW.rate, NOW());
       END IF;
   END //
   DELIMITER ;
   ```

   ------

   ### 🔎 **20. Eliminar productos sin relación a empresas**

   > "Como operador, quiero un trigger que elimine automáticamente productos sin relación a empresas."

   🧠 **Explicación:** Después de borrar la última relación entre un producto y una empresa (`companyproducts`), el trigger puede eliminar ese producto.

   ```
   DELIMITER //
   CREATE TRIGGER delete_product_if_no_company_relations
   AFTER DELETE ON companyproducts
   FOR EACH ROW
   BEGIN
       DECLARE company_product_count INT;
       SELECT COUNT(*) INTO company_product_count
       FROM companyproducts
       WHERE product_id = OLD.product_id;
       IF company_product_count = 0 THEN
           DELETE FROM products
           WHERE id = OLD.product_id;
       END IF;
   END //
   DELIMITER ;
   ```

   
