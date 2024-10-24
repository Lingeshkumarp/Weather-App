**Real-Time Weather Monitoring System Documentation**
**Objective:**
The real-time weather monitoring system fetches live weather data from the OpenWeatherMap API and displays key information such as temperature, humidity, wind speed, and weather conditions for a specified or current location. The system uses React.js for the frontend, Axios for API requests, and React Animated Weather for weather icons. This document covers the various modules in the system, their functions, and an explanation of the underlying code.
________________________________________
**Components of the System:**
1.	Forecast Component (Forecast.js)
2.	Current Location Weather Component (CurrentLocation.js)
3.	Service Worker Component (ServiceWorker.js)
4.	API Key Management (ApiKeys.js)
________________________________________
**1. Forecast Component (Forecast.js)**
This component handles the retrieval and display of weather information for a specified city using the OpenWeatherMap API.
Imports:
import React, { useState, useEffect } from "react";
import axios from "axios";
import apiKeys from "./apiKeys";
import ReactAnimatedWeather from "react-animated-weather";
•	React: Manages the state and rendering.
•	Axios: Facilitates HTTP requests to the weather API.
•	ReactAnimatedWeather: Displays animated weather icons based on the current conditions.
•	apiKeys: Stores the API key and base URL for easy reference.
States:
•	query: Stores the user-inputted city name.
•	error: Stores any error messages if the city is not found.
•	weather: Stores the weather data retrieved from the API.
Main Functionality:
•	Search Function: The search() function retrieves weather data for a specified city. It sends a request to the OpenWeatherMap API using Axios. If successful, the weather data is stored in the weather state; otherwise, an error is set.
const search = (city) => {
  axios
    .get(
      `${apiKeys.base}weather?q=${
        city != "[object Object]" ? city : query
      }&units=metric&APPID=${apiKeys.key}`
    )
    .then((response) => {
      setWeather(response.data);
      setQuery("");
    })
    .catch(function (error) {
      console.log(error);
      setWeather("");
      setError({ message: "Not Found", query: query });
    });
};
•	Weather Display: If the weather data is successfully retrieved, the component dynamically renders the temperature, humidity, visibility, wind speed, and more using data from the API.
<ul>
  {typeof weather.main != "undefined" ? (
    <div>
      <li>Temperature <span>{Math.round(weather.main.temp)}°c ({weather.weather[0].main})</span></li>
      <li>Humidity <span>{Math.round(weather.main.humidity)}%</span></li>
      <li>Visibility <span>{Math.round(weather.visibility)} mi</span></li>
      <li>Wind Speed <span>{Math.round(weather.wind.speed)} Km/h</span></li>
    </div>
  ) : (
    <li>{error.query} {error.message}</li>
  )}
</ul>
Weather Icons:
React Animated Weather is used to render icons based on the current weather conditions.
js
Copy code
<ReactAnimatedWeather
  icon={props.icon}
  color="white"
  size={112}
  animate={true}
/>
________________________________________
**2. Current Location Weather Component (CurrentLocation.js)**
This component retrieves the user's current location using the Geolocation API and displays real-time weather data based on the latitude and longitude.
Imports:
import React from "react";
import apiKeys from "./apiKeys";
import Clock from "react-live-clock";
import Forcast from "./forcast";
import ReactAnimatedWeather from "react-animated-weather";
Main Functionality:
•	Geolocation: The getPosition() function fetches the user's current latitude and longitude using the browser's Geolocation API. If successful, this data is passed to the getWeather() function to fetch weather data for that location.
getPosition = (options) => {
  return new Promise(function (resolve, reject) {
    navigator.geolocation.getCurrentPosition(resolve, reject, options);
  });
};
•	Weather Fetching: Once the latitude and longitude are retrieved, the getWeather() function sends an API request to OpenWeatherMap to fetch weather data based on the current location.
getWeather = async (lat, lon) => {
  const api_call = await fetch(
    `${apiKeys.base}weather?lat=${lat}&lon=${lon}&units=metric&APPID=${apiKeys.key}`
  );
  const data = await api_call.json();
  this.setState({
    lat: lat,
    lon: lon,
    city: data.name,
    temperatureC: Math.round(data.main.temp),
    temperatureF: Math.round(data.main.temp * 1.8 + 32),
    humidity: data.main.humidity,
    main: data.weather[0].main,
    country: data.sys.country,
  });
};
Weather Display:
•	The component displays the current temperature, city, country, and a weather icon based on real-time data.
<ReactAnimatedWeather
  icon={this.state.icon}
  color={defaults.color}
  size={defaults.size}
  animate={defaults.animate}
/>
•	The Clock component from react-live-clock is used to display the current time, updated in real-time.
________________________________________
**3. Service Worker Component (ServiceWorker.js)**
The service worker ensures that the app works offline and caches static assets for a better user experience.
Main Functionality:
•	Registration: Registers the service worker and ensures that assets are cached for offline use. It checks if the app is running in a production environment and if service workers are supported by the browser.
navigator.serviceWorker
  .register(swUrl)
  .then(registration => {
    registration.onupdatefound = () => {
      const installingWorker = registration.installing;
      if (installingWorker == null) {
        return;
      }
      installingWorker.onstatechange = () => {
        if (installingWorker.state === 'installed') {
          if (navigator.serviceWorker.controller) {
            console.log('New content is available and will be used when all tabs for this page are closed.');
          } else {
            console.log('Content is cached for offline use.');
          }
        }
      };
    };
  })
  .catch(error => {
    console.error('Error during service worker registration:', error);
  });
•	Cache Management: The service worker handles asset caching and ensures the app functions even without an internet connection.
________________________________________
**4. API Key Management (ApiKeys.js)**
This file stores the API key required for accessing the OpenWeatherMap API and the base URL of the API.
module.exports = {
  key: "YOUR_API_KEY",
  base: "https://api.openweathermap.org/data/2.5/",
};
