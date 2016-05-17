# UnityFrameworkExample
This repository is an example to work with Dependency Injection using UnityFramework with ASP.NET MVC with Multiple Repositories

#Background
<p>
This article explains the use of Unity Framework for Dependency Injection and takes a different approach for resolving dependency amongst classes and interfaces. Advantages of Dependency injection are simplified code and generate dependent object instances. We might be aware of registering types using C# but there is another approach as well. We can use the Configuration files to declare and register types and then Resolve in within our code. This article will help you understand it in a practical and clean approach. This article also covers a scenario of switching different Unity Configuration files based on requirements from the Business.
</p>

# Installation
We need an empty ASP.NET project(Web forms or MVC any is fine), and [Unity Framework](https://msdn.microsoft.com/en-us/library/ff660859(v=pandp.20).aspx). One can get the library/package from NuGet Package Manager. For this project, we only need one library.

# Usage
As we know we are using Unity for Dependency Injection, we need to register the types prior using. 
Here is the code to declare Unity in `configSections`

`<section name="unity" type="Microsoft.Practices.Unity.Configuration.UnityConfigurationSection, Microsoft.Practices.Unity.Configuration" />`

The next part is to declare the configuration source. There are two options, we can directly write down the code in `Web.Config` file or we can create a new Configuration File and call it in `Web.Config`

`<unity configSource="NewRepositoryUnity.config" />`

Assuming that we need to use multiple version of repositories, we can do that by creating a new `Unity Configuration Files` and then switching the `configSource` in `Web.Config` file. 

Now we need to start with the main code.
```xml
<unity xmlns="http://schemas.microsoft.com/practices/2010/unity">

  <!--Aliasing all types being used in the project to make the configuration file more readable-->
  <alias alias="CourierBusiness" type="UnityFrameworkExample.Business.CourierBusinessClass, UnityFrameworkExample" />
  <alias alias="UspsCourierRepository" type="Repository.NewRepositories.UspsCourierRepository, Repository" />
  <alias alias="FedexCourierRepository" type="Repository.NewRepositories.FedexCourierRepository, Repository" />
  <alias alias="ICourierRepository" type="Repository.Interfaces.ICourierRepository, Repository" />

  <container name="CourierContainer">

    <!--Registering Courier Interface with multiple classes which are implementing that particular interface-->
    <register type="ICourierRepository" mapTo="UspsCourierRepository" name="UspsCourierRepository"/>
    <register type="ICourierRepository" mapTo="FedexCourierRepository" name="FedexCourierRepository"/>

    <!-- Registering CourierBusinessClass to map the repository instance to Usps Repository-->
    <register type="CourierBusiness" name="UspsCourier">
      <constructor>
        <param name="courierRepository" type="ICourierRepository">
          <dependency name="UspsCourierRepository"/>
        </param>
      </constructor>
    </register>

    <!-- Registering CourierBusinessClass to map the repository instance to Fedex Repository-->
    <register type="CourierBusiness" name="FedexCourier">
      <constructor>
        <param name="courierRepository" type="ICourierRepository">
          <dependency name="FedexCourierRepository"/>
        </param>
      </constructor>
    </register>
  </container>
</unity>
```
Now lets go over the code.
The first part, we need to declare all the types we will be using during our demo. `alias` tag is used to declare the names and types of the objects. We specify the types with the help of `type` attribute and we mention the namespaces and assemblies of the objects.


Using the `container` tag, we can provide names to different containers and resolve only that particular container in our code. Using `name` attribute, we give name to our container.

Now we start with registering our objects to appropriately. In `register` tags, we mention the objects that we would like to map to. We usually mention the name of the Interfaces and map it to Classes the implements it. In case if an interface is being implemented by multiple classes, we can register it here in these tags and then resolve it. The first two lines are registering the Interfaces to `UspsCourierRepository` and `FedexCourierRepository`. The code will look for alias name `UspsCourierRepository` and then fetch the namespace and assembly from `alias` tag.

Along with Interfaces, we can also register classes for dependency injection. We have a constructor in `CourierBusiness` Class where we are injecting the Interface Repository. So we register the `CourierBusiness` Class in configuration file and provide name to it. As there are two implementations of the interface, we need to register the class two times with different Repository Implementations. As we are not using Custom Constructor, we define the parameter being passed and the type. The type is defined in the tag `dependency`

Now we've registered all our types, we now use it in our Code. We first need to load the container from `Web.Config`
```C#
IUnityContainer container = new UnityContainer();
container.LoadConfiguration("CourierContainer");
```

The code will lok for container name `CourierContainer` and load it. Once loaded successfully, we can then resolve the types.

```C#
  var uspsInstance = container.Resolve<CourierBusinessClass>("UspsCourier");
  ViewBag.uspsCourierPrice = uspsInstance.GetLatestPrices();
  ```
The above two lines of code will Resolve the type `CourierBusinessClass` and look for Registered type of name `UspsCourier`. Based on the name, application will create a new instance of `UspsCourierRepository` in the constructor of `CourierBusinessClass` and then when called the method and fetch results. 

```C#
  var fedexInstance = container.Resolve<CourierBusinessClass>("FedexCourier");
  ViewBag.fedexCourierPrice = fedexInstance.GetLatestPrices();
```
  The next two lines creates a new instance of CourierBusinessClass but this time, it will inject a new instance of `FedexCourierRepository` and execute the `GetLatestPrices()` inside it. 
  
  Below is the code in `CourierBusinessClass
```C#
  public class CourierBusinessClass
    {
        private ICourierRepository _courierRepostitory;

        public CourierBusinessClass(ICourierRepository courierRepository)
        {
            _courierRepostitory = courierRepository;
        }

        public string GetLatestPrices()
        {
            var message = _courierRepostitory.GetCourierPrices();
            return message;
        }
    }
```
    

Because of Unity Framework, we do not need to `New` the instances in the Business class, Unity does that for us and maps it to appropriate Class based on our configuration files.
    
## License
The project is open source and all the code is available online at Microsoft Website,
