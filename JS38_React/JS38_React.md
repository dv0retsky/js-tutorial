## Тема №36. React 🚀

## ❄️ Создаём приложение для прогноза погоды

<div align="center">
  <img alt="Project Demo" src="./JS38.jpeg" />
</div>

Прежде всего вам понадобится [**node.js**](https://nodejs.org/en/) и редактор кода. Откроем терминал и установим:

**React** - это JavaScript-библиотека для создания пользовательских интерфейсов. Основные концепции:

- Компоненты - reusable блоки UI
- `JSX` - синтаксис, позволяющий писать HTML-подобный код в JavaScript
- `Props` - свойства, передаваемые компонентам
- `State` - внутреннее состояние компонента
- `Hooks` - функции для использования состояния и других возможностей React

**Vite** - современный сборщик проектов, обеспечивает:

- Мгновенный запуск dev-сервера
- Быструю горячую перезагрузку
- Оптимизированную сборку

**API (Application Programming Interface)** - интерфейс для взаимодействия с внешними сервисами. В нашем случае - получение данных о погоде.

## 🌼 Практическая часть

Создаем новый проект `React` с `Vite`:

```bash
npm create vite@latest weather-app -- --template react
cd weather-app
npm install
```

Установка дополнительных зависимостей:

```bash
npm install axios
```

Структура проекта:

```bash
weather-app/
├── src/
│   ├── components/
│   │   ├── WeatherDisplay.jsx
│   │   └── CitySelector.jsx
│   ├── App.jsx
│   ├── main.jsx
│   └── App.css
├── index.html
└── package.json
```

**Основной компонент** `App.jsx`:

```js
import { useState, useEffect, useCallback } from 'react'
import './App.css'
import WeatherDisplay from './components/WeatherDisplay'
import CitySelector from './components/CitySelector'

const CITIES = [
  { name: "Москва" },
  { name: "Санкт-Петербург" },
  { name: "Новосибирск" },
  { name: "Екатеринбург" },
  { name: "Казань" }
]

function App() {
  const [selectedCity, setSelectedCity] = useState(CITIES[0])
  const [weatherData, setWeatherData] = useState(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)

  const API_KEY = '1a91b9730ac5419fa4f40859251811'

  const fetchMockWeather = useCallback(async (city) => {
    try {
      await new Promise(resolve => setTimeout(resolve, 500))
      
      const seasons = {
        winter: { min: -15, max: 0 },
        spring: { min: 5, max: 15 },
        summer: { min: 15, max: 25 },
        autumn: { min: 0, max: 10 }
      };
      
      const currentMonth = new Date().getMonth();
      let season;
      if (currentMonth >= 11 || currentMonth <= 1) season = 'winter';
      else if (currentMonth >= 2 && currentMonth <= 4) season = 'spring';
      else if (currentMonth >= 5 && currentMonth <= 7) season = 'summer';
      else season = 'autumn';
      
      const tempRange = seasons[season];
      const temp = Math.round(Math.random() * (tempRange.max - tempRange.min) + tempRange.min);
      
      const mockData = {
        name: city.name,
        main: {
          temp: temp,
          feels_like: temp - Math.round(Math.random() * 5),
          humidity: Math.round(Math.random() * 40 + 40),
          pressure: 1013 + Math.round(Math.random() * 20 - 10),
          temp_min: temp - Math.round(Math.random() * 3),
          temp_max: temp + Math.round(Math.random() * 3)
        },
        weather: [
          {
            description: ["ясно", "облачно", "пасмурно", "небольшой дождь", "снег"][Math.floor(Math.random() * 5)],
            icon: "//cdn.weatherapi.com/weather/64x64/day/113.png"
          }
        ],
        wind: {
          speed: (Math.random() * 8 + 2).toFixed(1)
        },
        sys: {
          country: "RU"
        }
      }
      
      setWeatherData(mockData)
    } catch (mockErr) {
    console.error('Ошибка в демо-данных:', mockErr)
    setError('Не удалось загрузить данные')
  }
}, [])

  const fetchWeather = useCallback(async (city) => {
    setLoading(true)
    setError(null)
    
    try {
      const response = await fetch(
        `https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=${city.name}&lang=ru`
      )
      
      if (!response.ok) {
        const errorData = await response.json()
        throw new Error(errorData.error?.message || `Ошибка ${response.status}: ${response.statusText}`)
      }
      
      const data = await response.json()
      
      const convertedData = {
        name: data.location.name,
        main: {
          temp: data.current.temp_c,
          feels_like: data.current.feelslike_c,
          humidity: data.current.humidity,
          pressure: data.current.pressure_mb,
          temp_min: data.current.temp_c - 2,
          temp_max: data.current.temp_c + 2
        },
        weather: [
          {
            description: data.current.condition.text,
            icon: data.current.condition.icon
          }
        ],
        wind: {
          speed: (data.current.wind_kph / 3.6).toFixed(1)
        },
        sys: {
          country: data.location.country
        }
      }
      
      setWeatherData(convertedData)
    } catch (err) {
      setError(err.message)
      console.error('Ошибка получения погоды:', err)
      
      await fetchMockWeather(city)
    } finally {
      setLoading(false)
    }
  }, [API_KEY, fetchMockWeather])

  useEffect(() => {
    if (selectedCity) {
      fetchWeather(selectedCity)
    }
  }, [selectedCity, fetchWeather])

  const handleRetry = () => {
    fetchWeather(selectedCity)
  }

  return (
    <div className="app">
      <header className="app-header">
        <h1>Прогноз погоды</h1>
        <p>React приложение с использованием WeatherAPI</p>
        <div className="api-status">
          🌤️ Работает через WeatherAPI.com
        </div>
      </header>

      <main className="app-main">
        <CitySelector 
          cities={CITIES}
          selectedCity={selectedCity}
          onCityChange={setSelectedCity}
        />

        {loading && (
          <div className="loading">
            <div className="spinner"></div>
            Загрузка данных о погоде...
          </div>
        )}

        {error && (
          <div className="error">
            <p>Ошибка: {error}</p>
            <button onClick={handleRetry} className="retry-button">
              Попробовать снова
            </button>
            <div className="fallback-notice">
              Если ошибка повторяется, будут показаны демо-данные
            </div>
          </div>
        )}

        {weatherData && !loading && (
          <>
            <WeatherDisplay weatherData={weatherData} />
            <div className="data-source">
              <small>
                Данные предоставлены WeatherAPI.com • 
                Последнее обновление: {new Date().toLocaleTimeString()}
              </small>
            </div>
          </>
        )}

      </main>
    </div>
  )
}

export default App
```

**Компонент выбора города** `CitySelector.jsx`:

```js
function CitySelector({ cities, selectedCity, onCityChange }) {
  return (
    <div className="city-selector">
      <h2>Выберите город:</h2>
      <div className="city-buttons">
        {cities.map((city, index) => (
          <button
            key={index}
            className={`city-button ${selectedCity.name === city.name ? 'active' : ''}`}
            onClick={() => onCityChange(city)}
          >
            {city.name}
          </button>
        ))}
      </div>
    </div>
  )
}

export default CitySelector
```

**Компонент отображения погоды** `WeatherDisplay.jsx`:

```js
function WeatherDisplay({ weatherData }) {
  if (!weatherData) return null

  const { name, main, weather, wind, sys } = weatherData
  const weatherInfo = weather[0]

  const getWeatherIcon = (iconCode) => {
    return `https://openweathermap.org/img/wn/${iconCode}@2x.png`
  }

  return (
    <div className="weather-display">
      <h2>
        Погода в {name}{sys?.country ? `, ${sys.country}` : ''}
      </h2>
      
      <div className="weather-card">
        <div className="weather-main">
          <img 
            src={getWeatherIcon(weatherInfo.icon)} 
            alt={weatherInfo.description}
            onError={(e) => {
              e.target.src = 'https://openweathermap.org/img/wn/01d@2x.png'
            }}
          />
          <div className="temperature">
            {Math.round(main.temp)}°C
          </div>
          <div className="weather-description">
            {weatherInfo.description}
          </div>
        </div>
        
        <div className="weather-details">
          <div className="detail-item">
            <span className="detail-label">Ощущается как:</span>
            <span className="detail-value">{Math.round(main.feels_like)}°C</span>
          </div>
          <div className="detail-item">
            <span className="detail-label">Влажность:</span>
            <span className="detail-value">{main.humidity}%</span>
          </div>
          <div className="detail-item">
            <span className="detail-label">Давление:</span>
            <span className="detail-value">{main.pressure} hPa</span>
          </div>
          <div className="detail-item">
            <span className="detail-label">Ветер:</span>
            <span className="detail-value">{wind.speed} м/с</span>
          </div>
          <div className="detail-item">
            <span className="detail-label">Мин. температура:</span>
            <span className="detail-value">{Math.round(main.temp_min)}°C</span>
          </div>
          <div className="detail-item">
            <span className="detail-label">Макс. температура:</span>
            <span className="detail-value">{Math.round(main.temp_max)}°C</span>
          </div>
        </div>
      </div>
    </div>
  )
}

export default WeatherDisplay
```

**Стилизация** `App.css`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
  min-height: 100vh;
  color: #333;
  display: flex;
  justify-content: center;
  align-items: center;
}

.app {
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.app-header {
  text-align: center;
  margin-bottom: 30px;
  color: white;
  width: 100%;
}

.app-header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.app-header p {
  font-size: 1.1rem;
  opacity: 0.9;
}

.city-selector {
  background: white;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  margin-bottom: 20px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.city-selector h2 {
  margin-bottom: 15px;
  color: #2d3436;
  text-align: center;
}

.city-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
  width: 100%;
}

.city-button {
  padding: 10px 20px;
  border: 2px solid #74b9ff;
  background: white;
  color: #74b9ff;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}

.city-button:hover {
  background: #74b9ff;
  color: white;
  transform: translateY(-2px);
}

.city-button.active {
  background: #0984e3;
  color: white;
  border-color: #0984e3;
}

.weather-display {
  background: white;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.weather-display h2 {
  color: #2d3436;
  margin-bottom: 20px;
  text-align: center;
  width: 100%;
}

.weather-card {
  display: flex;
  gap: 30px;
  align-items: center;
  flex-wrap: wrap;
  justify-content: center;
  width: 100%;
}

.weather-main {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.weather-main img {
  width: 100px;
  height: 100px;
}

.temperature {
  font-size: 3rem;
  font-weight: bold;
  color: #0984e3;
}

.weather-details {
  flex: 1;
  min-width: 250px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.weather-details p {
  margin-bottom: 10px;
  padding: 8px 0;
  border-bottom: 1px solid #eee;
  width: 100%;
  text-align: center;
}

.loading, .error, .demo-info {
  background: white;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
  margin: 20px 0;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  width: 100%;
}

.error {
  background: #ff7675;
  color: white;
}

.demo-info {
  background: #ffeaa7;
  color: #2d3436;
}

.api-status {
  background: rgba(76, 175, 80, 0.2);
  color: #2e7d32;
  padding: 8px 16px;
  border-radius: 20px;
  margin-top: 10px;
  font-size: 0.9rem;
  font-weight: 500;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(76, 175, 80, 0.3);
  text-align: center;
}

.error {
  background: #ffebee;
  color: #c62828;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
  margin: 20px 0;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  border-left: 4px solid #c62828;
}

.retry-button {
  background: #c62828;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  margin: 10px 0;
  font-weight: 500;
  transition: background 0.3s ease;
}

.retry-button:hover {
  background: #b71c1c;
}

.fallback-notice {
  margin-top: 10px;
  font-size: 0.9rem;
  opacity: 0.8;
  text-align: center;
}

.data-source {
  text-align: center;
  margin-top: 15px;
  color: rgba(255, 255, 255, 0.8);
  font-style: italic;
  width: 100%;
}

.data-source small {
  font-size: 0.8rem;
}

.weather-display {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 768px) {
  .app {
    padding: 10px;
  }
  
  .city-buttons {
    justify-content: center;
  }
  
  .weather-card {
    justify-content: center;
    text-align: center;
  }
  
  body {
    padding: 10px;
  }
}

@media (max-width: 480px) {
  .app-header h1 {
    font-size: 2rem;
  }
  
  .app-header p {
    font-size: 1rem;
  }
  
  .city-buttons {
    justify-content: center;
  }
  
  .city-button {
    flex: 1;
    min-width: 120px;
    font-size: 0.9rem;
  }
  
  body {
    align-items: flex-start;
    padding: 5px;
  }
}
```

Получение api реализовано через [**сайт**](https://www.weatherapi.com)

## 🐣 Пример реализации

Полный исходный код проекта доступен на GitHub:

🔗 https://github.com/dv0retsky/weather-app

## 👻 Задачи для практики

### 🧩 Задание 1: Добавление новой информации о погоде

**Задача:** Показать дополнительную информацию о погоде

**Что нужно сделать:**
- Добавить отображение "восхода солнца" и "заката"
- Показать "видимость" в километрах
- Добавить "облачность" в процентах

**Подсказки:**
- Данные уже приходят из API, нужно просто их отобразить
- Используйте компонент `WeatherDisplay`
- Добавьте новые строки в список с погодой

---

### 🧩 Задание 2: Изменение внешнего вида

**Задача:** Улучшить дизайн приложения

**Что нужно сделать:**
- Изменить цвета кнопок выбора города
- Добавить тени к карточке с погодой
- Сделать анимацию появления данных о погоде
- Изменить шрифты на более красивые

**Подсказки:**
- Работайте с файлом `App.css`
- Используйте CSS-переменные для цветов
- Для анимаций используйте `@keyframes`

---

### 🧩 Задание 3: Добавление нового города

**Задача:** Добавить возможность выбора дополнительных городов

**Что нужно сделать:**
- Добавить в список городов: "Ковров", "Владимир", "Иваново"
- Убедиться, что кнопки новых городов работают
- Проверить, что погода для новых городов отображается корректно

**Подсказки:**
- Найдите массив `CITIES` в коде
- Добавьте новые объекты в этот массив
- Для новых городов используйте английские названия

---

### 🧩 Задание 4: Сообщение о загрузке

**Задача:** Сделать индикатор загрузки более информативным

**Что нужно сделать:**
- Показывать название загружаемого города в сообщении о загрузке
- Добавить анимацию "пульсации" для кнопки выбранного города
- Показывать "Загружаем погоду для [название города]..."

**Подсказки:**
- Передавайте `selectedCity` в компонент загрузки
- Используйте CSS-анимацию `pulse`
- Измените текст в компоненте загрузки

---

### 🧩 Задание 5: Простая обработка ошибок

**Задача:** Улучшить сообщения об ошибках

**Что нужно сделать:**
- Показывать дружелюбное сообщение при ошибке сети
- Добавить подсказку, что делать при ошибке
- Сделать кнопку "Попробовать снова" более заметной

**Подсказки:**
- Измените текст в компоненте ошибки
- Добавьте CSS-стили для кнопки повтора
- Используйте понятные пользователю формулировки

---

<div align="center"> Made with ❤️ by <b>dv0retsky</b> </div>