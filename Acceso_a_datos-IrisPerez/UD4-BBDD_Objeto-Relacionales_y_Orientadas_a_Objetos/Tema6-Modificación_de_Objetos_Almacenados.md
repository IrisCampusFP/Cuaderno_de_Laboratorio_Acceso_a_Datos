## CE 4f: Modificar objetos almacenados

En este tema aprenderemos a actualizar y eliminar objetos estructurados en Oracle, PostgreSQL y JPA.

## 6.1 Actualización en Oracle

### Actualizar Atributos Simples

-- Actualizar un atributo
UPDATE productos p
SET p.precio = 999.99
WHERE p.codigo = 1;

-- Actualizar múltiples atributos
UPDATE productos p
SET p.precio = p.precio * 1.10,
    p.stock = p.stock + 10
WHERE p.nombre LIKE '%Laptop%';

-- Actualizar usando métodos
UPDATE productos p
SET p.precio = p.precio * 1.05
WHERE p.calcular_valor_stock() > 10000;

### Actualizar Objetos Anidados en Oracle

-- Actualizar campo de objeto anidado
UPDATE clientes c
SET c.direccion.ciudad = 'Barcelona',
    c.direccion.codigo_postal = '08001'
WHERE c.id = 1;

-- Reemplazar objeto anidado completo
UPDATE clientes c
SET c.direccion = tipo_direccion(
    'Paseo de Gracia 100',
    'Barcelona',
    '08008',
    'España'
)
WHERE c.id = 1;

### Actualizar Colecciones VARRAY en Oracle

-- Reemplazar VARRAY completo
UPDATE clientes_v2 c
SET c.telefonos = lista_telefonos(
    tipo_telefono('móvil', '666999888'),
    tipo_telefono('casa', '933334444'),
    tipo_telefono('trabajo', '934445555')
)
WHERE c.id = 1;

-- Nota: VARRAY no permite modificar elementos individuales
-- directamente. Hay que reemplazar toda la colección.

Advertencia

Los VARRAY en Oracle no permiten operaciones de agregar o eliminar elementos individuales. Para modificaciones parciales, hay que leer, modificar en PL/SQL, y guardar la colección completa.

### Actualizar NESTED TABLE en Oracle

-- Las NESTED TABLE son más flexibles
-- Insertar en nested table
INSERT INTO TABLE(
    SELECT c.emails FROM contactos c WHERE c.id = 1
) VALUES (tipo_email('nuevo', 'nuevo@email.com'));

-- Actualizar elemento de nested table
UPDATE TABLE(
    SELECT c.emails FROM contactos c WHERE c.id = 1
) e
SET e.email = 'actualizado@email.com'
WHERE e.tipo = 'personal';

-- Eliminar de nested table
DELETE FROM TABLE(
    SELECT c.emails FROM contactos c WHERE c.id = 1
) e
WHERE e.tipo = 'trabajo';

## 6.2 Actualización en PostgreSQL

### Actualizar Campos de Tipos Compuestos

-- Actualizar campo individual
UPDATE productos
SET datos.precio = 999.99
WHERE (datos).codigo = 1;

-- Reemplazar tipo compuesto completo
UPDATE productos
SET datos = ROW(1, 'Laptop HP Pro', 1099.99, 45, CURRENT_DATE)::tipo_producto
WHERE id = 1;

-- Actualizar campo anidado
UPDATE clientes
SET direccion.ciudad = 'Barcelona',
    direccion.codigo_postal = '08001'
WHERE id = 1;

-- Reemplazar objeto anidado completo
UPDATE clientes
SET direccion = ROW('Paseo de Gracia 100', 'Barcelona', '08008', 'España')::tipo_direccion
WHERE id = 1;

### Operaciones con Arrays en PostgreSQL

-- Agregar elemento al final
UPDATE clientes
SET telefonos = array_append(telefonos, '933445566')
WHERE id = 1;

-- Agregar al principio
UPDATE clientes
SET telefonos = array_prepend('900123456', telefonos)
WHERE id = 1;

-- Eliminar elemento por valor
UPDATE clientes
SET telefonos = array_remove(telefonos, '666111222')
WHERE id = 1;

-- Reemplazar elemento por posicion
UPDATE clientes
SET telefonos[1] = '600000000'
WHERE id = 1;

-- Concatenar arrays
UPDATE clientes
SET telefonos = telefonos || ARRAY['nuevo1', 'nuevo2']
WHERE id = 1;

-- Reemplazar array completo
UPDATE clientes
SET telefonos = ARRAY['666111111', '666222222']
WHERE id = 1;

### Actualización en Tablas Heredadas (PostgreSQL)

-- Actualizar solo en tabla hija
UPDATE libros
SET paginas = 500
WHERE isbn = '978-84-376-0494-7';

-- Actualizar atributos heredados desde tabla padre
-- (afecta a todas las tablas hijas que cumplan la condición)
UPDATE recursos
SET disponible = false
WHERE anio < 2000;

-- Actualizar SOLO en tabla padre
UPDATE ONLY recursos
SET disponible = false
WHERE anio < 2000;

## 6.3 Actualización con JPA

### Dirty Checking Automático

JPA detecta automáticamente los cambios en entidades gestionadas:

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// Encontrar entidad (queda en estado managed)
Producto p = em.find(Producto.class, 1L);

// Modificar atributos
p.setPrecio(p.getPrecio().multiply(new BigDecimal("1.10")));
p.setStock(p.getStock() + 10);

// No es necesario llamar a persist() ni merge()
// JPA detecta los cambios automáticamente

em.getTransaction().commit(); // Aquí se genera el UPDATE
em.close();

### Actualizar Objetos Embebidos

em.getTransaction().begin();

Cliente cliente = em.find(Cliente.class, 1L);

// Modificar campos del objeto embebido
cliente.getDireccion().setCiudad("Barcelona");
cliente.getDireccion().setCodigoPostal("08001");

// O reemplazar el objeto embebido completo
Direccion nuevaDireccion = new Direccion();
nuevaDireccion.setCalle("Paseo de Gracia 100");
nuevaDireccion.setCiudad("Barcelona");
nuevaDireccion.setCodigoPostal("08008");
nuevaDireccion.setPais("España");
cliente.setDireccion(nuevaDireccion);

em.getTransaction().commit();

### Actualizar Colecciones en JPA

em.getTransaction().begin();

Cliente cliente = em.find(Cliente.class, 1L);

// Agregar elemento
cliente.getTelefonos().add("933445566");

// Eliminar elemento
cliente.getTelefonos().remove("666111222");

// Limpiar y reemplazar
cliente.getTelefonos().clear();
cliente.getTelefonos().addAll(List.of("600111111", "600222222"));

em.getTransaction().commit();

### Merge de Entidades Detached

// Entidad obtenida en otra transacción (detached)
Producto productoDetached = // ...

em.getTransaction().begin();

// Modificar la entidad detached
productoDetached.setPrecio(new BigDecimal("1500.00"));

// Merge: copia los cambios a una entidad managed
Producto productoManaged = em.merge(productoDetached);

// Ahora productoManaged está gestionado por el EntityManager

em.getTransaction().commit();

### Actualización en Lote (Bulk Update)

em.getTransaction().begin();

// UPDATE masivo con JPQL
int actualizados = em.createQuery(
    "UPDATE Producto p SET p.precio = p.precio * 1.10 " +
    "WHERE p.stock > 0")
    .executeUpdate();

System.out.println("Productos actualizados: " + actualizados);

em.getTransaction().commit();

// Importante: El bulk update NO actualiza el caché de primer nivel
em.clear(); // Limpiar caché para ver cambios

## 6.4 Eliminación de Objetos

### Oracle

-- Eliminar objeto
DELETE FROM productos p WHERE p.codigo = 1;

-- Eliminar con condición en objeto anidado
DELETE FROM clientes c 
WHERE c.direccion.ciudad = 'Madrid';

-- Eliminar con condición en colección
DELETE FROM clientes_v2 c
WHERE EXISTS (
    SELECT 1 FROM TABLE(c.telefonos) t
    WHERE t.tipo = 'temporal'
);

### PostgreSQL

-- Eliminar con condición en tipo compuesto
DELETE FROM productos WHERE (datos).codigo = 1;

-- Eliminar con condición en campo anidado
DELETE FROM clientes WHERE (direccion).ciudad = 'Madrid';

-- Eliminar con condición en array
DELETE FROM clientes 
WHERE 'temporal' = ANY(telefonos);

### Eliminación con JPA

em.getTransaction().begin();

// Eliminar entidad gestionada
Producto p = em.find(Producto.class, 1L);
if (p != null) {
    em.remove(p);
}

em.getTransaction().commit();

// Eliminación en lote
em.getTransaction().begin();

int eliminados = em.createQuery(
    "DELETE FROM Producto p WHERE p.stock = 0")
    .executeUpdate();

em.getTransaction().commit();

### Cascada y OrphanRemoval

@Entity
public class Pedido {
    @Id
    @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "pedido", 
               cascade = CascadeType.ALL,
               orphanRemoval = true)
    private List<LineaPedido> lineas = new ArrayList<>();
}

// Con orphanRemoval = true
em.getTransaction().begin();

Pedido pedido = em.find(Pedido.class, 1L);

// Al eliminar de la coleccion, se elimina de la BD
pedido.getLineas().remove(0);

// Al limpiar, se eliminan todas las lineas huerfanas
pedido.getLineas().clear();

em.getTransaction().commit();

## 6.5 Resumen del Tema

Conceptos Clave

1. **Oracle**: Actualiza campos anidados con notación punto, VARRAY requiere reemplazo completo.
    
2. **PostgreSQL**: Funciones `array_append`, `array_remove` para manipular arrays.
    
3. **JPA**: Dirty checking detecta cambios automáticamente en entidades managed.
    
4. `em.merge()` sincroniza entidades detached con la base de datos.
    
5. `orphanRemoval = true` elimina entidades huérfanas automáticamente.
    
6. Bulk updates con JPQL son más eficientes pero no actualizan el caché.