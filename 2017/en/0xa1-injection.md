# A1:2017 Inyección

| Agentes de amenaza/Vectores de ataque | Debilidades de seguridad           | Impactos técnicos               |
| -- | -- | -- |
| Nivel de acceso \| Explotabilidad FÁCIL | Prevalencia COMÚN \| Detección PROMEDIO | Impacto SEVERO \| Específico de la aplicación/negocio |
| Casi cualquier fuente de información puede ser un vector de inyección, incluyendo usuarios, parámetros, web services internos y externos, y todo tipo de usaurios. [Las fallas de inyección](http://www.owasp.org/index.php/Injection_Flaws) ocurren cuando un atacante puede enviar información hostil a un intérprete. | Estas fallas son muy comunes, particularmente en el código antiguo. Se encuentran frecuentemente en las consultas SQL, LDAP, XPath o NoSQL, comandos de SO, intérpretes de XML, encabezados de SMTP, lenguages de expresión o consultas ORM. Estas fallas son fáciles de descubrir al examinar el código. Scanners y "fuzzers" pueden ayudar a los atacantes a encontrar fallas de inyección. | Una inyección puede causar pérdida o corrupción de datos, pérdida de responsabilidad, o negación de acceso. Algunas veces una inyección puede llevar al compromiso total del servidor. El impacto en el negocio depende de las necesidades de protección de su aplicación y datos. |

## ¿Soy vulnerable?

Una aplicación es vulnerable a ataques cuando:

* La información provista por el usuario no es validada, filtrada o sanitizada por la aplicación.
* Información no confiable se usa directamente en consultas dinámicas o llamadas no parametrizadas al intérprete sin usar exclusiones dependientes del contexto.
* Información no confiable es usada en parámetros de búsqueda ORM de tal forma que los parámetros de la búsqueda sean evaluados para incluir registros sensibles o todos los registros.
* Información no confiable es usada directamente o concatenada, de tal forma que las sentencias SQL o comandos incluyan tanto información de estructura como información hostil en consultas dinámicas, comandos o procedimientos almacenados.

Algunos de los ataques más comunes de inyección son SQL, comandos del SO, ORM, LDAP o Lenguaje de Expresiones (EL) o inyección OGNL. El concepto es idéntico entre todos los intérpretes. Las organizaciones pueden incluir herramientas SAST y DAST en el pipeline CI/CD para alertar si nuevo código o código existente tiene vulnerabilidades de inyección previo a su puesta en producción. La revisión de código fuente manual y automatizada es el mejor método para detectar si se es vulnerable a inyecciones, seguido de cerca por escaneos DAST de todos los parámetros, campos, encabezados, cookies, entradas de datos JSON y XML.

## ¿Cómo prevengo los ataques de inyección?

Para prevenir la inyección, se requiere mantener los datos no confiables separados de los comandos y las consultas.

* La opción preferida es usar una API segura la cual evite el uso de intérpretes por completo o provea una interfaz parametrizada, o migrar para usar ORMs o Entity Framework. Al ser parametrizados, los procedimientos almacenados igualmente pueden introducir fallas de inyección SQL si PL/SQL o T-SQL concatena consultas y datos, o ejecuta información no confiable con el comando EXECUTE IMMEDIATE o exec().
* La validación de entradas positiva o de "lista blanca" también se recomienda, pero no es una defensa integral dado que muchas aplicaciones requieren caracteres especiales en sus entradas, como en cajas de texto o APIs para aplicaciones móviles.
* Para el caso de consultas dinámicas residuales, excluya caracteres especiales usando la sintaxis de exclusión específica para ese intérprete. El [OWASP Java Encoder](https://www.owasp.org/index.php/OWASP_Java_Encoder_Project) y librerías similares proveen estas rutinas de exclusión. La estructura SQL como los nombres de tablas, nombres de columna y demás no pueden ser excluida, por lo que la estructura proporcionada por el usuario es peligrosa. Esta es una falla común en software de generación de reportes.
* Use LIMIT y otros controles de SQL en las consultas para prevenir la divulgación masiva de registros en caso de un ataque de inyección SQL.

## Ejemplos de Escenarios de Ataque

**Escenario #1**: La aplicación usa datos no confiables en la construcción de la siguiente instrucción SQL vulnerable:

```
String query = "SELECT * FROM accounts WHERE custID='" + request.getParameter("id") + "'";
```

**Escenario #2**: De manera similar, si una aplicación confía ciegamente en el framework puede resultar en consultas que aún son vulnerables (ej. Hibernate Query Language (HQL)):

```
Query HQLQuery = session.createQuery("FROM accounts WHERE custID='" + request.getParameter("id") + "'");
```

En ambos casos, al atacante modificar el parametro 'id' en su navegador para enviar:  ' or '1'='1. Por ejemplo:
* `http://example.com/app/accountView?id=' or '1'='1`

Esto cambia el significado de ambas consultas regresando todos los registro de la tabla "accounts". Ataques más peligrosos pueden modificar datos o incluso invocar procedimientos almacenados.

## Referencias

### OWASP

* [OWASP Proactive Controls: Parameterize Queries](https://www.owasp.org/index.php/OWASP_Proactive_Controls#2:_Parameterize_Queries)
* [OWASP ASVS: V5 Input Validation and Encoding](TBA)
* [OWASP Testing Guide: SQL Injection](https://www.owasp.org/index.php/Testing_for_SQL_Injection_(OTG-INPVAL-005)), [Command Injection](https://www.owasp.org/index.php/Testing_for_Command_Injection_(OTG-INPVAL-013)), [ORM injection](https://www.owasp.org/index.php/Testing_for_ORM_Injection_(OTG-INPVAL-007))
* [OWASP Cheat Sheet: SQL Injection Prevention](https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet)
* [OWASP Cheat Sheet: Injection Prevention in Java](https://www.owasp.org/index.php/Injection_Prevention_Cheat_Sheet_in_Java)
* [OWASP Cheat Sheet: Query Parameterization](https://www.owasp.org/index.php/Query_Parameterization_Cheat_Sheet)
* [OWASP Cheat Sheet: Command Injection Defense](https://www.owasp.org/index.php/Command_Injection_Defense_Cheat_Sheet)

### Externas

* [CWE-77 Command Injection](https://cwe.mitre.org/data/definitions/77.html)
* [CWE-89 SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
* [CWE-564 Hibernate Injection](https://cwe.mitre.org/data/definitions/564.html)
* [CWE-917 Expression Language Injection](https://cwe.mitre.org/data/definitions/917.html)
* [PortSwigger: Server-side template injection](https://portswigger.net/knowledgebase/issues/details/00101080_serversidetemplateinjection)
