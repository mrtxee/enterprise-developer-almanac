# SQL — Structured Query Language
SQL — это предметно ориентированный язык `**DSL**`**.**
## SQL joins
`**SELECT * FROM A INNER[|LEFT|RIGHT|FULL OUTER] JOIN B ON A.key=B.key [WHERE *IS NULL*]**`
- `LEFT / RIGHT / INNER` присоединяет к пересечению левую часть, правую, либо только пересечение
- `FULL OUTER` внешние части
- `WHERE A[|B] IS [NOT] NULL` когда хотим исключить или включить часть
![[attachments/Untitled 6 4.png|Untitled 6 4.png]]

## [[DSL]] — Domain-specific language
### DDL – Data Definition Language
- CREATE – используется для создания объектов базы данных;
- ALTER – используется для изменения объектов базы данных;
- DROP – используется для удаления объектов базы данных.
### DML – Data Manipulation Language
- SELECT – осуществляет выборку данных;
- INSERT – добавляет новые данные;
- UPDATE – изменяет существующие данные;
- DELETE – удаляет данные.
### DCL – Data Control Language
- GRANT – предоставляет пользователю или группе разрешения на определённые операции с объектом;
- REVOKE – отзывает выданные разрешения;
- DENY– задаёт запрет, имеющий приоритет над разрешением.
### TCL – Transaction Control Language
- BEGIN TRANSACTION – служит для определения начала транзакции;
- COMMIT TRANSACTION – применяет транзакцию;
- ROLLBACK TRANSACTION – откатывает все изменения, сделанные в контексте текущей транзакции;
- SAVE TRANSACTION – устанавливает промежуточную точку сохранения внутри транзакции.