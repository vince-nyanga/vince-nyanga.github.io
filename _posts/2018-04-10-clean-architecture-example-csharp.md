---
title: Clean Architecture Example C#
date: 2018-04-10
tags: [C#, Clean Architecture]
---

When I was still in university I failed a job interview because I did not know the architecture I was using in my project (embarrassing). Since that day I decided to learn about different software architectures and use them in all my projects.

Uncle Bob’s [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) is one of the architectures that I’ve come across and I fell in love with it. The reason why I like it is that it helps create software that is testable, independent of frameworks, UI or databases. It also enforces a rule that source code dependencies point inwards (see image below). This means that something in the inner layer cannot know about something in the outer layer.

<figure>
<img src="{{ site.baseurl }}/images/clean-architecure.jpeg" alt="Clean architecture">
<figcaption>Clean Architecture, courtesy: Uncle Bob</figcaption>
</figure>

For detailed explanation of the clean architecture check out this [post](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) by Uncle Bob.

### Let's build something
We are going to create a simple .Net console application that shows the weather of a given location. The complete source code is available on [GitHub](https://github.com/vince-nyanga/CleanArchitectureExample).
The application has three layers — domain, data and presentation layers. The image below shows how the layers depend on each other. A layer can only know about a layer below it as shown by the arrows.

<figure>
<img src="{{ site.baseurl }}/images/clean-architecure-arrows.png" alt="Clean architecture diagram">
<figcaption>The three layers used in the weather app</figcaption>
</figure>

#### Domain Layer
This is the bottom layer in our application. It contains the entities, use-cases and interfaces. Code in this layer is as abstract and generic as possible. I simply defines how the application should work. More ‘meat’ will be added in layers above.

#### Entities
These are the blocks that build our application and are not affected by any external forces, that is, external changes won’t change them. In our weather app we only have one entity — `WeatherEntity`.

```csharp
    public class WeatherEntity
    {
        public string Description { get; set; }
        public double MinTemperature { get; set; }
        public double MaxTemperature { get; set; }

    }
```

#### UseCases
These are application specific business rules. Each use case (aka interactor) implements a single business use case in our system, for instance, get weather for a location. In the weather application all use cases implement a base interface `IRequestHandler`.

```csharp
    internal interface IRequestHandler<in TRequest, out TResponse>
    {
       TResponse Handle(TRequest data); 
    }
```
Now we create out use case that gets the weather for a given location.

```csharp
    public class GetWeatherInteractor: IRequestHandler<string,WeatherEntity>
    {
        private readonly IRepository _repository;

        public GetWeatherInteractor(IRepository repository)
        {
            _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        }


        public WeatherEntity Handle(string data)
        {
            if (string.IsNullOrEmpty(data))
            {
                throw new ArgumentNullException(nameof(data));
            }

            return _repository.GetWeather(data);
        }
    }
```

#### Interfaces
The interfaces in our domain layer include the repository from which our application data will be loaded. It’s just a simple interface with one method.

```csharp
    public interface IRepository
    {
        WeatherEntity GetWeather(string cityName);
    }
```

The use of an interface ensures that our code is testable and also that our domain layer is independent of any implementation of the repository. The code extract below shows how handy interfaces are when writing tests.

```csharp
    [TestFixture]
    public class UseCaseTests
    {
       
        private readonly WeatherEntity _weather = new WeatherEntity
        {
            Description = "Cloudy",
            MaxTemperature = 25,
            MinTemperature = 12
        };

        private GetWeatherInteractor _interactor;

        [OneTimeSetUp]
        public void SetUp()
        {
            var mock = new Mock<IRepository>();
            mock.Setup(repo => repo.GetWeather("Harare")).Returns(_weather);
            _interactor = new GetWeatherInteractor(mock.Object);
        }

        [Test]
        public void TestGetWeather()
        {
            var result = _interactor.Handle("Harare");
            Assert.NotNull(result);
            Assert.AreEqual(result.MaxTemperature, _weather.MaxTemperature);
            Assert.AreEqual(result.MinTemperature, _weather.MinTemperature);
            Assert.AreEqual(result.Description, _weather.Description);
        }
    }
}
```

#### Data layer
This layer is responsible for handling our application data. It contains implementations of the data interfaces defined in the domain layer. The data layer may also have entities specific to it as determined by the data providers used as seen in this example. Let’s create the implementation of our repository.

```csharp
    public class WeatherRepository: IRepository
    {
        private readonly IApi _api;
        private readonly IMapper<WeatherData, WeatherEntity> _mapper;

        public WeatherRepository(IApi api,IMapper<WeatherData, WeatherEntity> mapper)
        {
            _api = api ?? throw new System.ArgumentNullException(nameof(api));
            _mapper = mapper ?? throw new System.ArgumentNullException(nameof(mapper));
        }


        public WeatherEntity GetWeather(string cityName)
        {
            if (string.IsNullOrEmpty(cityName))
            {
                throw new System.ArgumentNullException(nameof(cityName));
            }

            return _mapper.MapFrom(_api.GetWeatherData(cityName));
        }
    }
```

#### Mappers
You may have noticed that our weather repository takes in an IMapper interface. This is a simple interface that maps one object to another. The reason behind the use of this interface is that the data we get in the data layer may not look anything close to the entities defined in the domain layer. This then means we will have to convert the data we get into our domain entities. In our case we get our weather data from [openweathermap](https://openweathermap.org/) and it comes back as a completely different object from our domain entity hence the need for a mapper.

```csharp
    public class WeatherDataEntityMapper: IMapper<WeatherData, WeatherEntity>
    {
        public WeatherEntity MapFrom(WeatherData input)
        {
            if (input == null)
            {
                throw new System.ArgumentNullException(nameof(input));
            }

            return new WeatherEntity
            {
                MinTemperature = input.Main.MinTemperature,
                MaxTemperature = input.Main.MaxTemperature,
                Description = input.WeatherList[0].Description
            };
        }
    }
```

#### Presentation layer
This is the upper most layer in out application. It is responsible wiring everything together as well as interact with the user. In this layer we initialize and hook up all the objects. We will use dependency injection using NInject. Note this layer may have an architecture of its own (MVP, MVC, MVVM etc.). Since everything is decoupled the presentation layer can be a mobile app, web app or anything really without us changing anything in the layers below it. In this case it is a simple console application.
Let’s create a module for our dependency injection.

```csharp
    public class WeatherModule: NinjectModule
    {
        public override void Load()
        {
            var apiKey = ConfigurationManager.AppSettings["API_KEY"];
            Bind<IMapper<WeatherData, WeatherEntity>>().To<WeatherDataEntityMapper>();
            Bind<IApi>().To<OpenWeatherApi>().WithConstructorArgument("apiKey",apiKey);
            Bind<IRepository>().To<WeatherRepository>();
            Bind<GetWeatherInteractor>().ToSelf().InSingletonScope();
        }
    }
```

Let’s get the weather :)

```csharp
    internal class Program
    {
        private static void Main(string[] args)
        {
           
            IKernel kernel = new StandardKernel(new WeatherModule());
            var useCase = kernel.Get<GetWeatherInteractor>();
            const string city = "Johannesburg";
            var weather = useCase.Handle(city);
            Console.WriteLine($"Weather for {city}:");
            Console.WriteLine($"Description: {weather.Description}");
            Console.WriteLine($"Min Temp: {weather.MinTemperature}°C");
            Console.WriteLine($"Max Temp: {weather.MaxTemperature}°C");
            Console.ReadLine();

        }
    }
```

### Summary
In this post we have created a simple console application that gets the weather for a given city using The Clean Architecture. This architecture has many advantages including building software that is testable and independent of frameworks.

The complete source code is available on [GitHub](https://github.com/vince-nyanga/CleanArchitectureExample). Once again, thanks for reading.
