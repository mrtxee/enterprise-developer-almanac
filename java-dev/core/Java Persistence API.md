Одна из имплементаций → [[hibernate]]
# JPA — Jakarta Persistence API
Jakarta Persistence API (JPA; ранее Java Persistence API) — спецификация API Jakarta EE, предоставляет возможность сохранять в удобном виде Java-объекты в базе данных. Существует несколько реализаций этого интерфейса, одна из самых популярных использует для этого Hibernate. JPA описывает спецификацию для ORM.
# Entity
Entity (Сущность) — POJO-класс, связанный с БД с помощью аннотации (`@Entity`) или через XML. К такому классу предъявляются следующие требования:
- Должен иметь пустой конструктор (`public` или `protected`)
- Не может быть вложенным, интерфейсом или enum
- Не может быть `final` и не может содержать `final`полей/свойств
- Должен содержать хотя бы одно `@Id`поле
При этом entity может:
- Содержать непустые конструкторы
- Наследоваться и быть наследованным
- Содержать другие методы и реализовывать интерфейсы
Entities могут быть связаны друг с другом (один-к-одному, один-ко-многим, многие-к-одному и многие-ко-многим)
Описан в спецификации JPA
## persistent class — сохраняемый класс
Если мы хотим, чтобы экземпляры (объекты) Java-класса в сохранялся в БД, то мы называем их “сохраняемые классы” — persistent class. Persistent class-ы являются **POJO** (Plain Old Java Object) и должны отвечать следующим критериям
- Все классы должны иметь ID для простой идентификации наших объектов в БД и в Hibernate. Это поле класса соединяется с первичным ключём (primary key) таблицы БД.
- Все POJO – классы должны иметь конструктор по умолчанию (пустой).
- Все поля POJO – классов должны иметь модификатор доступа private иметь набор getter-ов и setter-ов в стиле JavaBean.
- POJO – классы не должны содержать бизнес-логику.
# Аннотации JPA
Заметка с аннотациями и примерами JPA
|   |   |
|---|---|
|@Entity|javax.persistence.Entity|
|@Table|javax.persistence.Table|
|@Column|javax.persistence.Table|
|@Id|javax.persistence.Id|
|@GeneratedValue|javax.persistence.GeneratedValue|
|@Version|javax.persistence.Version|
|@OrderBy|javax.persistence.OrderBy|
|@Transient|javax.persistence.Transient|
|@Lob|javax.persistence.Lob|
|@PersistenceContext|javax.persistence.PersistenceContext|
|@Temporal|javax.persistence.TemporalType|
|@Embedded|javax.persistence.Embedded|
|Аннотации для связи ассоциаций||
|@OneToOne|javax.persistence.OneToOne|
|@ManyToOne|javax.persistence.ManyToOne|
|@OneToMany|javax.persistence.OneToMany|
|@ManyToMany|javax.persistence.ManyToMany|
|@PrimaryKeyJoinColumn|javax.persistence.PrimaryKeyJoinColumn|
|@JoinColumn|javax.persistence.PrimaryKeyJoinColumn|
|@JoinTable|javax.persistence.JoinTable|
|@MapsId|javax.persistence.MapsId|
|Аннотации наследования||
|@Inheritance|javax.persistence.Inheritance|
|@DiscriminatorColumn|javax.persistence.DiscriminatorColumn|
|@DiscriminatorValue|javax.persistence.DiscriminatorValue|
|Аннотации запросов||
|@NamedQueries|javax.persistence.NamedQueries|
|@NamedQuery|javax.persistence.NamedQuery|
|@SqlResultSetMapping|javax.persistence.SqlResultSetMapping|
|@EntityResult|javax.persistence.EntityResult|
|Аннотации Hibernate||
|@Audited|org.hibernate.envers.Audited|
|@NotAudited|org.hibernate.envers.NotAudited|
|@Type|org.hibernate.annotations.Type|
**@Entity** — Указывает, что данный бин (класс) является сущностью.
@Entity
public class Cat implements Serializable {
...
}

---
**@Table** — указывает на имя таблицы, которая будет отображаться в этой сущности.
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
...
}

---
**@Column** — указывает на имя колонки, которая отображается в свойство сущности.
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@Column(name = "catName")
private String name;
...
}

---
**@Id** — id колонки
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@Id
@Column(name = "id")
private int id;
...
}

---
**@GeneratedValue** — указывает, что данное свойство будет создаваться согласно указанной стратегии.
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@Id
@GeneratedValue(strategy = IDENTITY) //AUTO, SEQUENCE, TABLE
@Column(name = "id")
private int id;
...
}

---
**@Version** — управление версией в записи сущности. При изменении записи увеличится на 1.
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@Version
@Column(name = "version")
private int version;
...
}

---
**@OrderBy** — указание сортировки. В примере множество кошек будет отсортировано по имени по возрастанию.
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@OrderBy("firstName asc")
private Set catsSet;
...
}

---
**@Transient** — указывает, что свойство не нужно  
записывать. Значения под этой аннотацией не записываются в базу данных  
(так же не участвуют в сериализации). static и final переменные  
экземпляра всегда transient.  
@Entity
@Table(name = "cat_table")
public class Cat implements Serializable {
@Transient
public boolean isNew() {
return id == null;
}
...
}

---
**@Lob** — указание на большие объекты.
**@PersistenceContext** — указывает на зависимость EntityManager в контейнере
@Service("jpaCatService")
@Repository
@Transactional
public class CatServiceImpl implements CatService {
@PersistenceContext
private EntityManager em;
...
}

---
**@Temporal** — применяется к полям или свойствам с  
типом java.util.Date и java.util.Calendar. Например, если в БД время  
сохраняется как sql.Date, то чтобы использовать дату из java.util.Date  
указываем эту аннотацию.  
public class Cat implements Serializable {
private java.util.Date birthDate; //in DB schema uses sql.Date in column BIRTH_DATE.
@Temporal(TemporalType.DATE)
@Column(name = "BIRTH_DATE")
public Date getBirthDate() {
return this.birthDate;
}
...
}

---
**@Embeddable** и **@Embedded** («встроенный») — Определяет класс, экземпляры которого хранятся как неотъемлемая часть исходного объекта. Каждый из **@Embedded** экземпляров сопоставляется с таблицей базы данных сущности.
@Embeddable public class Address {
protected String street;
protected String city;
protected String state;
@Embedded protected Zipcode zipcode;
}
@Embeddable public class Zipcode {
protected String zip;
protected String plusFour;
}

---
Аннотации для ассоциаций: один к одному, один ко многим, многие ко многим и др.
**@OneToOne** — указывает на связь между таблицами «один к одному».
**orphanRemoval** — позволяет удалять объекты сироты. При удалении родительского объекта удаляется и дочерний.
**@OneToMany(mappedBy=»customer», orphanRemoval=»true»)**
**mappedBy** — обратная сторона связи сущности. Поле под  
этим атрибутом не сохраняется как часть исходной сущности в базе данных,  
но будет доступна по запросу. (см. ниже **@JoinColumn**)
cascade = **CascadeType.ALL** — означает, что операция, например, записи должна распространяться и на дочерние таблицы.
@Entity
@Table(name = "contactDetail")
public class ContactDetail implements Serializable {
@Id
@Column(name = "id")
@GeneratedValue
private int id;
@OneToOne
@MapsId
@JoinColumn(name = "contactId")
private Contact contact;
...
}
@Entity
@Table(name = "contact")
public class Contact implements Serializable {
@Id
@Column(name = "ID")
@GeneratedValue
private Integer id;
@OneToOne(mappedBy = "contact", cascade = CascadeType.ALL)
private ContactDetail contactDetail;
....
}

---
В этом примере контакт содержит лишь ссылку на таблицу с детальной  
информацией. Детали контакта будут доступны по запросу к таблице  
contactDetail.  
**@PrimaryKeyJoinColumn** — главный ключ для ассоциированной сущности с таким же ключом
**@JoinColumn** — применяется когда внешний ключ  
находится в одной из сущностей. Может применяться с обеих сторон  
взаимосвязи. Но рекомендуется применять в сущности, которая является  
владельцем физической информации (обычно сторона **@ManyToOne**). ManyToOne (часто) является стороной-владельцем в двунаправленных связях и таким образом противоположная сторона использует **@OneToMany**(mappedBy=..).
_Пример (более распространенное использование mappedBy и JoinColumn)_
@Entity
public class Troop {
@OneToMany(mappedBy="troop")
public Set getSoldiers() {
...
}
@Entity
public class Soldier {
@ManyToOne
@JoinColumn(name="troop_fk")
public Troop getTroop() {
...
}

---
Менее распространенный вариант (нужно еще менять свойства **insertable** и **updatable**). Такое решение менее оптимизировано, но будет работать:
@Entity
public class Troop {
@OneToMany
@JoinColumn(name="troop_fk") //we need to duplicate the physical information
public Set getSoldiers() {
...
}
@Entity
public class Soldier {
@ManyToOne
@JoinColumn(name="troop_fk", insertable=false, updatable=false)
public Troop getTroop() {
...
}

---
**@JoinTable** — указывает на связь с таблицей
**@ManyToOne** — указывает на связь многие к одному.
**@OneToMany** — указывает на связь один ко многим. Применяется с другой стороны от сущности с @ManyToOne
@Entity
@Table(name = "contact")
public class Contact implements Serializable {
private Set contactTelDetails = new HashSet();
//...
@OneToMany(mappedBy = "contact", cascade=CascadeType.ALL, orphanRemoval=true)
public Set getContactTelDetails() {
return this.contactTelDetails;
}
....
}
@Entity
@Table(name = "contact_tel_detail")
public class ContactTelDetail implements Serializable {
private Contact contact;
//...
@ManyToOne
@JoinColumn(name = "CONTACT_ID")
public Contact getContact() {
return this.contact;
}
}

---
**@ManyToMany** — связь многие ко многим.
@Entity
@Table(name = "contact")
public class Contact implements Serializable {
private Set hobbies = new HashSet();
//...
@ManyToMany
@JoinTable(name = "contact_hobby_detail",
joinColumns = @JoinColumn(name = "CONTACT_ID"),
inverseJoinColumns = @JoinColumn(name = "HOBBY_ID"))
public Set getHobbies() {
return this.hobbies;
}
....
}
@Entity
@Table(name = "hobby")
public class Hobby implements Serializable {
private Set contacts = new HashSet();
//...
@ManyToMany
@JoinTable(name = "contact_hobby_detail",
joinColumns = @JoinColumn(name = "HOBBY_ID"),
inverseJoinColumns = @JoinColumn(name = "CONTACT_ID"))
public Set getContacts() {
return this.contacts;
}
}

---
**@MapsId** — связывает одну колонку с другой. Работает с **@Id** и **@EmbeddedId**.
@Entity
@Table(name="student")
public class Student implements Serializable {
@Id
@Column(name="sl_id")
private int slNum;
@MapsId
@OneToOne
@JoinColumn(name = "std_id")
private Person student;
@Column(name="location")
private String location;
public Student(int slNum,Person student, String location){
this.slNum=slNum;
this.student=student;
this.location=location;
}
@Entity
@Table(name="person")
public class Person implements Serializable {
@Id
@Column(name="p_id")
private int id;
@Column(name="first_name")
private String firstName;
@Column(name="last_name")
private String lastName;
..
}

---
**@Inheritance** — наследование. Для понимания аннотации необходимо обратиться к руководству Hibernate User Guide (Inheritance)
**@DiscriminatorColumn**
**@DiscriminatorValue**
@Entity
@Table(name="EJB_PROJECT")
@Inheritance(strategy=JOINED, discriminatorValue="P")
@DiscriminatorColumn(name="PROJ_TYPE")
public class Project implements Serializable {
...
@Id()
@Column(name="PROJECT_ID", primaryKey=true)
public Integer getId() {
return id;
}
...
}

---
# Аннотации запросов
**@NamedQueries** — внутри указывается список именованных запросов.
**@NamedQuery** — имя именованного запроса и сам запрос.
**@SqlResultSetMapping** — куда будет собран результат.
**@EntityResult** — указание сущности в которой будет сконструирован результат.
# Аннотации Hibernate
@Audited — Включает аудит сущности (отслеживание версий).
@NotAudited — указывает, что изменения этого свойства отслеживать не  
нужно. По умолчанию Hibernate пытается отслеживать изменения в  
ассоциациях.  
@Type указывает на тип свойства
@Entity
@Audited
@Table(name = "contact_audit")
@NamedQueries({
@NamedQuery(name="ContactAudit.findAll",
query="select c from Contact c"),
@NamedQuery(name="ContactAudit.findById",
query="select distinct c from Contact c left join fetch c.contactTelDetails t left join fetch c.hobbies h where c.id = :id"),
@NamedQuery(name="ContactAudit.findAllWithDetail",
query="select distinct c from Contact c left join fetch c.contactTelDetails t left join fetch c.hobbies h")
})
@SqlResultSetMapping(
name="contactAuditResult",
entities=@EntityResult(entityClass=ContactAudit.class)
)
public class ContactAudit implements Auditable<String, Long>, Serializable {
.....
@Column(name="CREATED_DATE")
@Type(type="org.joda.time.contrib.hibernate.PersistentDateTime")
public DateTime getCreatedDate() {
return createdDate;
}
@OneToMany(mappedBy = "contact", cascade=CascadeType.ALL, orphanRemoval=true)
@NotAudited
public Set getContactTelDetails() {
return this.contactTelDetails;
}
}