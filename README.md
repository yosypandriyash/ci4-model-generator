# CodeIgniter 4.x Model Generator

Este comando ha sido creado para ser usado exclusivamente en el framework PHP Codeigniter en su versión 4 y en adelante y bases de datos relacionales mysql. Por el momento no funciona con bases de datos SQL Server (no ha sido probado).

Este comando se encarga de analizar la estructura de las tablas de su base de datos y posteriormente de generar todos los modelos de dichas tablas (Modelos anémicos). Estos modelos, dispondrán de los métodos Getter y Setter para cada campo de su tabla así como de métodos de validación independientes para cada campo.

# Parámetros de configuración
Los parámtros de configuración permiten especificarle al generador de modelos cómo se deberan de generar estos, tanto su ubicación como su nombre entre otros aspectos.
Dicha configuración es tomada de la variable **$defaultOptions**:
Esta variable contiene los siguientes campos:

**cleanOnly**: (true | false) Establecer a true si lo que se quiere es solo limpiar los modelos existentes actualmente en el directorio de modelos establecido

**templates**: (array) Configuración prestablecida de las plantillas que se usarán para generar todos los modelos.

**keepOnClear**: Ficheros que se desean conservar en el egenerado de modelos.
*Base*: (array) Determina que ficheros se deben conservar en el directorio Base/... al generar los modelos.
*'Default'*:   (array)  Determina que ficheros se deben conservar en el directorio Models/... al generar los modelos.
Se puede usar el asterisco '*' para conservar todos los ficheros existentes tanto para la configuración del directorio Models como de Base

**overwriteBaseModel**: (true|false) Al generar los modelos se puede especificar si se quiere o no sobrescribir el fichero *BaseModel.php*. Esto es útil para mantener métodos, funciones y propiedades añadidas por el usuario en dicho fichero.

**Path:** (array) Especifica en que directorios se escribiran los modelos resultantes. También especifica el namespace de dichos ficheros.

**namespace**: (string) Define el namespace del que parten todos los demás modelos generados

**extension['base']**: Extensión de los modelos ubcados en el directorio Models/Base/

**extension['default']**: Extensión de los modelos ubcados en el directorio Models/

use
generatedModelExtendsFrom
defaultReturnType
defaultUseSoftDeletes
injectForeignTablesDependencies
useTimestamps
createdField
updatedField
deletedField

El comando, creará, (entre otros) dos ficheros para cada tabla de su base de datos.

# Fichero modelsMapper.php #
Este fichero se genera con cada invocación del comando. Contiene la ubicación (namespace) de todos los modelos generados. Es importante no modificar la configuración de este fichero.


# Fichero base "BaseModel.php"

Este fichero es el fichero "Master" del que heredarán todos los demás ficheros que se generen dinámicamente. Se trata de la clase BaseModel, la cual, entre otros, dispone de todos los métodos necesarios para poder trabajar con los modelos finales que se autogeneren con este comando. Estos métodos se pueden usar en cualquier clase que herede de nuestro BaseModel y son:

**getLastSqlError()**: (string) Éste metodo devolverá el último error que se haya producido en el servidor MYSQL tras haber ejecutado alguna consulta errónea o que ésta haya algún error. (Se recomienda usar para debugeo, en entorno test, ya que puede devolver información sensible de nuestra base de datos o de consultas realizadas)

**getLastInsertionId()**: (int) Devuelve el último id insertado o actualizado de nuestra tabla. Si no se ha insertado o actualizado previamente, devolverá null.

**getValidationErrors()**: (array) Devuelve los errores interceptados por nuestro validador de atributos. Si no se ha intercepta ningún error, devolverá un array vacío por lo que se podría usar if (!getValidationErrors()) para verificar que los datos a que se han validado son correctos aunque se recomienda validar independientemente cada atributo de manera específica con las reglas de negoco de cada aplicación.

**getTableForeignRelations($key || null)**: (array) Devuelve un array con las relaciones de claves foráneas de nuestra tabla con otras tablas. Si se especifica un $key, devolverá únicamente las relaciones externas de ese campo.

**getPrimaryKeyValue()**: (multi) devuelve el valor (no el nombre) del atributo definido mediante $primaryKey de nuestro objeto.

**getMapper($key || null)**: (array | string) Devuelve el mapper con la converión realizada entre las propiedades de la clase con las propiedades o campos de nuestra tabla MYSQL. Si se especifica $key, solo devolverá la traducción de la propiedad $key.

Métodos de obtención de datos:

**getFirst($options = [])**: (Object) Obtiene, según el filtro en \$options el primer resultado de la consulta.

**getOneById($id)**: (Object) Obtiene, usando el atributo $primaryKey del modelo, un único resultado que coincida con el $id del argumento.

**getAll(\$options = [])**: (Array | Object): Similar al getWhere(\$options = []) propio de codeigniter, busca resultados basándose en las opciones declaradas en la variable $options. Si $options no es declarado, busca todos los resultados sin aplicar filtros de búsqueda. Este método devuelve un array de objetos de la clase correspondiente si existe más de un resultado para el filtro o un array vacío si no existe ningún resultado.

**getOneByKey($key, $value)**: (Object): Devuelve un único objeto buscando por clave - valor.

**getAllByKey($key, $value)**: (Array): Similar getOneByKey() pero sin limitar el número de resultados. Siempre devuelve un array de objetos como resultado.

**query($mysqlQuery, $convertToModel = true)**: (Array): Permite ejecutar una consulta sql personalizada devolviendo como resultado un array de objectos de la misma clase sobre la cual se invoca dicho método. Si se quiere ejecutar una consulta multitabla o una consulta con nombres de columna distintos a los nombres de parámetros del modelo generado (en la que no se podrían matchear los atributos de la clase con los datos del array) se deberá usar un segundo argumento $convertToModel = false. De este modo, el método devolverá un array que contendra un array por cada registro de la consulta y por tanto, no se podrá usar ninguno de los métodos de la clase sobre el resultado de la consulta.

**save($data = [])**: (bool)
Permite actualizar o crear un registro nuevo en base de datos.
Dependiendo de si el objeto sobre el que se invoca este método tiene definido un valor en su atributo de clave primaria, esta función, creará (insert) un registro nuevo en base de datos en el caso de que no tenga el valor en dicha propiedad y actualizará (update) en base de datos en caso de que sí la tenga. Esto permite crear un objeto desde cero desde nuestro controlador sin necesidad de buscar un resultado sobre el que modificar previamente. Por ejemplo:

$newUser = $userModel = new UserModel();

$newUser->setName('Pepito')->save(); // Creará un nuevo registro
$userModel->getOneById(12)->setName('abc')->save() // Actualizará el campo name en el regitro cuyo id (clave primaria) sea '12'

Tras este método podemos invocar al método getLastInsertionId() para obtener el nuevo id insertado (INSERT) o el id del registro afectado (UPDATE)

Esta clase BaseModel también incluye más métodos de caracter privado y que sirven para funcionamiento interno óptimo, así como también el núcleo del proceso de validación de campos, etc.

> El usuario puede añadir, si lo desea, más métodos y atributos en esta
> clase para enriquecer el funcionamiento de esta clase y las demás clases herederas de esta.


# Fichero [table]BaseModel.php:

Suponiendo que tengamos la tabla users, nuestro comando, creará, en primer lugar el fichero UsersBaseModel, que, por defecto estará ubicado en App\Models\Base\ y que extenderá en herencia de la clase BaseModel citada anteriormente. Además, esta clase incluirá los métodos citados anteriormente (getters, setters y validadores).
También, dentro de este primer fichero se añadirán los propios atributos de clase Model de codeigniter ($table, $primaryKey, $returnType, etc.).

> Es recomendable no añadir atributos ni métodos en esta clase ya que
> por lo general, y por defecto, esta clase es pensada para almacenar
> los atributos de la tabla así como sus métodos de seteado y
> validación, por lo que si en un momento dado, añadimos un nuevo campo
> en la tabla y queremos regenerar el modelo (para tenerlo actualizado
> acorde a la tabla), todos los atributos extra que hayamos añadido se
> eliminaran.
>
> Para este caso, es recomendable usar la clase %tableName%Model, donde
> podremos guardar nuestros métodos y atributos personalizados sin temor
> a que se pierdan tras reinvocar nuestro comando de generación de
> modelos.

**protected $mapper & getMapper();**
Se añadirá también en este primer caso, un atributo de clase (protegido) para cada campo de la tabla, así como un mapper general de la tabla para convertir o referenciar los atributos creados con cada campo de nuestra tabla mysql.

**protected $validationRules**
Por último, en este primer fichero, se añadirá también una variable protegida denominada $validatioRules, la cual, se genera automáticamente para cada campo de la tabla y contiene la configuración de cada campo de la tabla para poder usarse con los validadores anteriormente mencionados. De esta manera, en nuestro modelo Users, disponiendo del campo $name, podremos invocar, por ejemplo, el método $userModel->validateName('nuevo nombre'); y este método nos devolverá true o false según las reglas de validación existentes en nuestra variable $validationRules.

**\$tableForeignRelations**
Esta variable ubicada en {tableName}BaseModel sirve para relacionar algunas propiedades el modelo actual (de su tabla correpondiente) con atributos de otras tablas (Modelos BaseModel). Esto se realiza automáticamente en el proceso de generación de modelos y permite ejecutar y obtener resultados enriquecidos con datos de varias tablas relacionadas usando un mismo método.
Para ello, estableceremos la propiedad protected **\$injectForeignTablesDependencies** a true en nuestro {tableName}Model.php

Por ejemplo:
$PostsModel->getOneByKey('posts_id', 123)->getUserId()->getName();
Nótese que para obtener el resultado relacionado, se llama al método getUserId ya que es el que contiene el "id" relacionado / "clave foránea".


# Fichero [table]Model.php:
Es el fichero que contiene la clase que el usuario deberá usar para invocar tanto a los métodos propios de dicha clase, como los métodos y propiedades de la clase BaseModel respectiva a esta e igualmente los métodos y atributos de la clase BaseModel genérica.
Esta clase se debe usar para almacenar en ella métodos personalizados y propiedades personalizadas.

> Por lo general, esta clase solo se crea una única vez y, a menos que
> se modifique la configuración general del generador, este fichero no
> seeliminará ni se sobrescribirá en futuras ejecuciones de este comando
> (Revisar cinfiguracion)

BaseModel > {tableName}BaseModel > {tableName}Model



(la configuración de su base de datos será tomada desde App/Config/Database o por el contrario del fichero .env si este dispone de dicha configuraión)
