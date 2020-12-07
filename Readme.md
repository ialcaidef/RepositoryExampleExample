#Demostración de como usar el patrón de diseño Repository

El patrón repositorio consiste en separar la lógica que recupera los datos y los asigna a un modelo de entidad de la lógica de negocios que actúa sobre el modelo, esto permite que la lógica de negocios sea independiente del tipo de dato que comprende la capa de origen de datos, en pocas palabras un repositorio media entre el dominio y las capas de mapeo de datos, actuando como una colección de objetos de dominio en memoria (M. Fowler).

En el proyecto EntityFrameworkExample crearemos las Clases que implementan las Interfaces del repositorio genérico dentro de la carpeta ##Respositories##

### Clase IRepository.cs
~~~
    public interface IRepository
    {
        IEnumerable<Person> GetPeople();
        void CreatePerson();
        void UpdatePerson(int id);
        void DeletePerson(int id);
    }
~~~

### Clase MyRepository.cs
~~~
    public class MyRepository : IRepository
    {
        private PersonContext _context;

        public MyRepository(PersonContext context)
        {
            _context = context;
        }

        public IEnumerable<Person> GetPeople()
        {
            return _context.People.ToList();
        }
        public void CreatePerson()
        {
            _context.Add(new Person() { FirstName = "Robert ", LastName = "Berends", City = "Birmingham", Address = "2632 Petunia Way" });
            _context.SaveChanges();
        }

        public void UpdatePerson(int id)
        {
            var person = _context.People.SingleOrDefault(m => m.PersonId == id);
            person.FirstName = "Brandon";
            _context.Update(person);
            _context.SaveChanges();
        }

        public void DeletePerson(int id)
        {
            var person = _context.People.SingleOrDefault(m => m.PersonId == id);
            _context.People.Remove(person);
            _context.SaveChanges();
        }
    }
~~~
En esta implementación crearíamos un repositorio base que contendría los métodos del típico CRUD (Create, Read, Update, Delete). que luegp serán utilizados por el controlador

~~~
    public class PersonController : Controller
    {
        private IRepository _repository;

        public PersonController(IRepository repository)
        {
            _repository = repository;
        }

        public IActionResult Index()
        {
            //return View();
            var list = _repository.GetPeople();
            return View(list);
        }

        public IActionResult Create()
        {
            _repository.CreatePerson();
            return RedirectToAction(nameof(Index));
        }

        public IActionResult Edit(int id)
        {
            _repository.UpdatePerson(id);
            return RedirectToAction(nameof(Index));
        }

        public IActionResult Delete(int id)
        {
            _repository.DeletePerson(id);
            return RedirectToAction(nameof(Index));
        }
    }
~~~

###Conclusiones
Es posible que el concepto de “Repository” esté pervertido hoy en día, con respecto al concepto inicial. Pero se ha ido adaptando a los diferentes progresos de las plataformas y los lenguajes.

Hay gente que no está de acuerdo con el nombre de “Repository” y que piensan que es simplemente un DAO. Aunque creo que no es del todo ni lo uno ni lo otro, simplemente coge algo de cada uno.

Los objetos de este tipo son muy útiles para desacoplar la aplicación y separar los datos del negocio de verdad. Además nos proveen de una forma probada y que funciona para resolver este problema, sin que tengamos que inventar nada nuevo. Son fáciles de probar y una abstracción muy útil para proyectos grandes. Así que si de verdad se implementa de forma que contenga la información acerca del almacenamiento, quitando esa responsabilidad al resto de componentes, es una opción muy válida.

Pero evidentemente y como pasa con todos los patrones de diseño, hay que ceñirse a las necesidades reales y no caer en los típicos antipatrones de aplicarlo cuando no es necesario o aplicarlo de una forma que no resulta del todo correcta.
https://www.developerro.com/2013/01/07/patrones-de-diseo-repository/
