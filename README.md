# DogCeoApi

Dog CEO Rest API Documentation
Overall, the service retrieves dog breed images, either from a cache or from a vendor API, and handles the caching and storage of fetched images in the database.
DogAPI Project
Project.cs Class
This code sets up a web application using ASP.NET Core to provide endpoints for retrieving images of dog breeds. Here's a summary of what each part does:
1.	Configuration Setup: Sets up Serilog for logging, configuring it to log to both the console and a file.
2.	Dependency Injection Configuration: Configures dependency injection for the application, including setting up the data context using Entity Framework Core to connect to a SQL Server database, and registering various services such as repositories and API clients.
3.	Swagger Configuration: Configures Swagger UI for API documentation.
4.	Endpoint Configuration: Defines a single endpoint for retrieving images of dog breeds. It accepts parameters for the breed and sub-breed, if applicable, and invokes the LookUpImageAsync method to fetch the image URL. If an error occurs during the lookup process, it logs the error and returns a 404 Not Found response with the error message.
DogCeoApi.cs Class
This code defines a service (`DogCeoAPI`) responsible for interacting with the Dog Ceo API to fetch random images of dog breeds. Here's a summary:

1. **Constructor**: Initializes the service with the required dependencies, including a logger (`ILogger`), application settings (`IAppSettings`), and a REST client (`IRestClient`).

2. **GetDogBreedPictureAsync Method**: This method asynchronously fetches a random image URL for a specified dog breed. It constructs a REST request using RestSharp, sends the request to the Dog Ceo API, and then deserializes the JSON response into a `DogCeoResponse` object using Newtonsoft.Json.

   - If the request is unsuccessful (e.g., non-200 status code), it logs the error using the provided logger (`ILogger`) and throws an exception with details from the API response.
   - If there are issues with deserialization, it logs the error and throws a `JsonSerializationException`.
   - If any unexpected error occurs during the process, it logs the error and throws a generic exception.

   All caught exceptions are logged using the provided logger (`ILogger`).

DogBreedRepository.cs Class
This code defines a repository (`DogBreedRepository`) responsible for handling interactions with a data source (Microsoft SQL Database) for dog breeds. Here's a summary:
1. **Constructor**: Initializes the repository with a logger (`ILogger`) and a context for accessing the database (`GoodDogContext`).
2. **InitializeCache Method**: Initializes a cache (`DogBreedCache`) with dog breed data from the database. This cache is a thread-safe dictionary (`ConcurrentDictionary`) that stores dog breeds by their names.
3. **InsertAsync Method**: Inserts a new dog breed into the database. It adds the breed to the database context and saves changes asynchronously. If successful, it updates the cache with the new breed data.
4. **UpdateCache Method**: Updates the cache with new dog breed data. If the breed exists in the cache, it updates it; otherwise, it logs a warning.
5. **Retrieve Method**: Retrieves a dog breed from the cache by its name. If the breed exists in the cache, it returns it; otherwise, it returns null.
The repository provides methods for inserting and retrieving dog breeds from the database while maintaining a cache for efficient access. It also ensures thread safety during cache initialization and updates.

ImageLookUp.cs Class

This code defines a service (`ImageLookUp`) responsible for looking up images of dog breeds. Here's a summary:
1. **Constructor**: Initializes the service with dependencies, including a logger (`ILogger`), a repository for dog breeds (`IDogBreedRepository`), and an API client for fetching dog breed images (`IDogCeoAPI`).
2. **LookUpImageAsync Method**: Asynchronously looks up a dog breed image. It first attempts to retrieve the image from the cache using the repository. If the image is not found in the cache, it fetches the image from the vendor API (`_dogCeoAPI.GetDogBreedPictureAsync`) and then downloads the image using `DownloadImage` method. After fetching the image, it creates a new `DogBreed` object with the breed name, image URL, and image bytes, inserts it into the repository, and returns the new `DogBreed` object.

3. **DownloadImage Method**: Asynchronously downloads an image from a given URL using `HttpClient`. If the download is successful, it returns the image bytes; otherwise, it throws an exception with the HTTP status code.

ConsoleDogBreed Project
This code is a console application (`ConsoleDogBreed`) that allows users to retrieve images of dog breeds from a web API. This application provides a simple interactive interface for users to view dog breed images fetched from a web API. Users can cycle through different sub-breeds and choose to open the images in their web browser.

1. **Main Method**: 
   - Loads configuration settings from an `appsettings.json` file using `IConfigurationRoot`.
   - Reads base URL, default breed, and default sub-breed from the configuration.
   - Asks the user if they want to open the retrieved images in a browser.
   - Enters a loop where it prompts the user to enter a sub-breed or uses the default sub-breed. Then, it constructs a request URL and sends an HTTP request to fetch the image.
   - If the user chooses to open the image in a browser (`yesNoKey`), it calls the `OpenUrlInBrowser` method.
   - Handles HTTP request exceptions and unexpected errors.

2. **LoadConfiguration Method**: Loads configuration settings from the `appsettings.json` file.

3. **OpenUrlInBrowser Method**: Opens a URL in the default web browser using the `Process.Start` method.

Business Unit Test Project

This project defines unit tests (ImageLookUpTests) for the ImageLookUp class, which is responsible for fetching images of dog breeds. Here's a summary:
 Setup Method: Initializes the unit tests by creating mocks for the dependencies (IDogBreedRepository, IDogCeoAPI, and ILogger<ImageLookUp>), and instantiates an ImageLookUp object with these mocks.
LookUpImageAsync_WithExistingBreedInCache_ShouldReturnCachedBreed Method: Tests the scenario where a breed image is found in the cache. It sets up the Retrieve method of the repository mock to return a cached breed, then verifies that the method returns the expected breed with the correct properties.
 LookUpImageAsync_WithNonExistingBreedInCache_ShouldFetchFromAPIAndReturnBreed Method: Tests the scenario where a breed image is not found in the cache and needs to be fetched from the API. It sets up the GetDogBreedPictureAsync method of the API mock to return a specific image URL, simulates a cache miss, and verifies that the method returns the expected breed with the correct properties. It also verifies that the breed is inserted into the repository.
 LookUpImageAsync_WhenApiCallFails_ShouldThrowException Method: Tests the scenario where the API call fails. It sets up the GetDogBreedPictureAsync method of the API mock to throw an exception, simulates a cache miss, and verifies that the method throws an exception.
Overall, these tests cover various scenarios for fetching dog breed images, including cache hits, cache misses, and API call failures, ensuring that the ImageLookUp class behaves correctly under different conditions.

