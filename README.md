# Mood-Based Song Recommender 🎵

A FastAPI application that recommends songs based on your mood and the current weather in any city. Uses LLM (Gemini) for intelligent mood-weather matching and Last.fm for song recommendations.

## Features ✨

- **Weather Integration**: Real-time weather data from OpenWeatherMap API
- **Smart Mood Matching**: LLM-powered mood-weather correlation
- **Song Recommendations**: Music suggestions from Last.fm API
- **Persistent Caching**: 30-minute weather data caching with JSON persistence
- **Fallback Logic**: Rule-based matching when LLM is unavailable
- **REST API**: Simple HTTP endpoints for easy integration

## Project Structure 📁

```
mood_song_recommender/
├── main.py                      # FastAPI application & endpoints
├── weather_api.py              # Weather data fetching & caching
├── mood_weather_matcher.py     # Rule-based mood-weather matching
├── llm_mood_weather_matcher.py # LLM-powered mood-weather matching
├── song_recommender.py         # Song recommendations via Last.fm
├── config.py                   # API keys & configuration
├── requirements.txt            # Python dependencies
├── test_app.py                 # Test suite
├── test_llm_matcher.py         # LLM matcher tests
├── weather_cache.json          # Persistent weather cache (auto-generated)
└── README.md                   # This file
```

## Setup 🚀

### Prerequisites
- Python 3.8+
- API keys for:
  - [OpenWeatherMap](https://openweathermap.org/api) (free)
  - [Last.fm](https://www.last.fm/api) (free)
  - [Google Gemini](https://aistudio.google.com/app/apikey) (free tier available)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd mood_song_recommender
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   # Windows
   .\venv\Scripts\activate
   # macOS/Linux
   source venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure API keys**
   Edit `config.py` and add your API keys:
   ```python
   OPENWEATHER_API_KEY = 'your_openweather_api_key'
   LASTFM_API_KEY = 'your_lastfm_api_key'
   GEMINI_API_KEY = 'your_gemini_api_key'
   LLM_MODEL = 'gemini-2.5-flash'
   ```

5. **Run the application**
   ```bash
   uvicorn main:app --reload
   ```

The API will be available at `http://127.0.0.1:8000`

## API Endpoints 🌐

### Core Endpoint
- **GET** `/recommend/?mood={mood}&city={city}`
  - Returns song recommendation based on mood and weather
  - Example: `GET /recommend/?mood=happy&city=Bangalore`

### Cache Management
- **GET** `/cache/stats` - View cache statistics
- **DELETE** `/cache/clear` - Clear all cached weather data
- **GET** `/health` - Health check with cache stats

## Usage Examples 💡

### Basic Song Recommendation
```bash
curl "http://127.0.0.1:8000/recommend/?mood=happy&city=Hyderabad"
```

**Response:**
```json
{
  "message": "The weather in Hyderabad is clear sky. It matches your happy mood! Here's a song for you: Bohemian Rhapsody by Queen"
}
```

### Check Cache Status
```bash
curl "http://127.0.0.1:8000/cache/stats"
```

**Response:**
```json
{
  "cache_statistics": {
    "total_entries": 3,
    "valid_entries": 2,
    "expired_entries": 1,
    "cache_duration_minutes": 30
  },
  "message": "Cache has 2 valid entries and 1 expired entries"
}
```

## Testing 🧪

Run the test suite:
```bash
python -m pytest test_app.py test_llm_matcher.py -v
```

## How It Works 🔧

1. **Weather Fetching**: Gets current weather for the specified city
2. **Mood-Weather Matching**: Uses LLM to determine if mood matches weather
3. **Song Recommendation**: Fetches appropriate songs from Last.fm based on mood
4. **Caching**: Stores weather data for 30 minutes to reduce API calls
5. **Fallback**: Uses rule-based matching if LLM is unavailable

## Caching System 💾

- **Duration**: 30 minutes per city
- **Storage**: JSON file (`weather_cache.json`)
- **Persistence**: Survives server restarts
- **Management**: Clear cache via API endpoint

## Error Handling 🛡️

- Graceful fallback to rule-based matching if LLM fails
- API error handling for weather and music services
- Cache validation with automatic expiration
