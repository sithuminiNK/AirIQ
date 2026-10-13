# 🌍AirIQ — Next-Hour PM2.5 Air Quality Forecasting System

**IT3051 – Fundamentals of Data Mining | Mini Project 2026 | SLIIT**

AirIQ is an end-to-end data mining and machine learning web application developed to estimate next-hour PM2.5 concentrations at selected air-quality monitoring stations across Beijing.

The application combines a web dashboard, a Python backend, live environmental data services, automated feature construction, and a trained Random Forest regression pipeline to produce a next-hour forecast.

> **Project purpose:** Demonstrate how data mining and machine learning can be integrated into an application that supports environmental monitoring and better-informed decisions.

## Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [Key Features](#key-features)
- [Technology Stack](#technology-stack)
- [System Architecture](#system-architecture)
- [Dataset](#dataset)
- [Feature Engineering](#feature-engineering)
- [Machine Learning Model](#machine-learning-model)
- [Model Evaluation](#model-evaluation)
- [Application Workflow](#application-workflow)
- [API Endpoints](#api-endpoints)
- [Installation and Setup](#installation-and-setup)
- [Testing and Monitoring](#testing-and-monitoring)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)
- [Team Members](#team-members)
- [Data Sources and Acknowledgements](#data-sources-and-acknowledgements)
- [Disclaimer](#disclaimer)

## Project Overview

Air quality can change over time and vary between locations. Monitoring current conditions is valuable, but an estimate of near-future conditions may provide additional time to review developing concerns.

AirIQ explores this problem by forecasting PM2.5 concentrations one hour ahead for selected monitoring stations in Beijing.

The user selects a station through the dashboard. The backend retrieves recent environmental observations, prepares the model features, passes them through a saved machine learning pipeline, and returns the prediction to the dashboard.

The project demonstrates the integration of:

- Data mining and historical data analysis
- Time-based feature engineering
- Machine learning model development and evaluation
- Live environmental data acquisition
- Backend API development
- Frontend dashboard development
- Model integration and inference

## Problem Statement

Traditional monitoring focuses on observed or current environmental conditions. Decision-makers may also benefit from an estimate of what conditions could look like in the near future.

The project investigates whether historical air-quality patterns and recent environmental observations can be used to estimate next-hour PM2.5 concentrations.

AirIQ aims to make this estimate accessible through a user-friendly dashboard instead of requiring users to process raw data or enter numerous model features manually.

## Objectives

1. Explore historical air-quality patterns across multiple monitoring stations.
2. Prepare time-based and environmental features for next-hour forecasting.
3. Develop and evaluate machine learning models for PM2.5 regression.
4. Integrate a trained model into a backend application.
5. Retrieve recent environmental data automatically through external services.
6. Display forecasts and relevant context in a web dashboard.
7. Demonstrate how short-term forecasts could support human decision-making.

## Key Features

### Station-based forecasting

Users can select from 12 supported Beijing monitoring stations.

### Automated environmental data collection

The backend retrieves recent weather and air-quality observations from Open-Meteo services. Users do not need to enter all model variables manually.

### Automated feature construction

The backend prepares temporal, lag, and rolling-statistic features in the format expected by the trained model.

### Random Forest prediction

The saved preprocessing and Random Forest regression pipeline is loaded by the backend to generate the next-hour PM2.5 estimate.

### Forecast dashboard

The interface displays the predicted PM2.5 value, relevant timestamps, measurement unit, and supporting environmental information available in the application.

### Input validation and API monitoring

The backend validates the selected station and records technical API activity in server logs, helping the team check whether requests are reaching the application successfully.

## Technology Stack

| Component | Technologies |
|---|---|
| Frontend | React, TypeScript, Vite |
| Backend | Python, FastAPI |
| Machine learning | scikit-learn, Random Forest regression |
| Model persistence | Joblib |
| Live environmental data | Open-Meteo Weather API, Open-Meteo Air Quality API |
| Data processing | Historical air-quality observations, temporal features, lag and rolling statistics |

## System Architecture

```text
                         User
                           |
                           v
              React + TypeScript + Vite
                    Web Dashboard
                           |
                     HTTP Request
                           |
                           v
                    FastAPI Backend
                           |
               +-----------+-----------+
               |                       |
               v                       v
       Open-Meteo Weather       Open-Meteo Air
             API                 Quality API
               |                       |
               +-----------+-----------+
                           |
                           v
             Merge Data by Timestamp
                           |
                           v
              Live Feature Construction
                           |
                           v
           Saved Preprocessing Pipeline
                           |
                           v
              Random Forest Regressor
                           |
                           v
                Next-Hour PM2.5
                    Prediction
                           |
                           v
                 FastAPI Response
                           |
                           v
                  AirIQ Dashboard
```

The architecture separates the user interface, backend processing, live-data retrieval, feature construction, and model inference.

## Dataset

The historical model-development work used the **Beijing Multi-Site Air-Quality dataset** from the UCI Machine Learning Repository.

The project presentation describes the dataset as containing:

- **420,768 hourly records**
- **12 monitoring stations**
- Historical observations covering **2013–2017**

The historical data contains pollutant and meteorological observations, including PM2.5, PM10, SO₂, NO₂, CO, O₃, temperature, pressure, dew point, rainfall, wind direction, and wind speed.

The historical dataset was used for data exploration, feature engineering, model development, and evaluation.

**Important distinction:** The current application retrieves live prediction inputs from Open-Meteo. The live data source is different from the historical dataset used during model development.

## Feature Engineering

The Random Forest pipeline expects a specific set of model features. The backend constructs these features from recent environmental observations and the selected station.

### Environmental features

- PM2.5
- PM10
- CO
- NO₂
- SO₂
- O₃
- Temperature
- Surface pressure
- Dew point
- Rainfall
- Wind speed
- Wind direction

### Categorical and calendar features

- Monitoring station
- Wind direction
- Season
- Year, month, day, and hour
- Day of the week
- Weekend indicator

### Cyclical time features

Hour-of-day information is also represented using sine and cosine features. This allows the model to represent the cyclical nature of time, where hours near midnight are close to each other.

### Lag features

Historical PM2.5 values are represented using lags of:

- 1 hour
- 2 hours
- 3 hours
- 6 hours
- 12 hours
- 24 hours

### Rolling statistics

The feature construction includes:

- Rolling means over 3, 6, and 24 hours
- Rolling standard deviations over 6 and 24 hours

Lag and rolling features are intended to use past and current observations rather than future information. This is important for avoiding future-information leakage during forecasting.

## Machine Learning Model

The final selected model integrated into the application is a **Random Forest regression pipeline**.

Random Forest combines predictions from multiple decision trees to produce a regression estimate. In AirIQ, the model estimates the PM2.5 concentration for the next hour.

The trained pipeline is saved as a Joblib artifact and loaded by the backend when the application starts. This allows the application to reuse the fitted preprocessing and model configuration during prediction requests.

The project also evaluated other regression approaches during model development, including:

- Linear Regression
- Decision Tree Regression
- Random Forest Regression
- Histogram-based Gradient Boosting Regression

The selected Random Forest model was the best-performing machine learning model among the evaluated machine learning algorithms during walk-forward validation, according to the system description.

## Model Evaluation

The system description reports the following historical evaluation results for Random Forest:

| Metric | Reported result |
|---|---:|
| Mean walk-forward validation R² | 0.9231 |
| 2017 holdout MAE | 14.74 µg/m³ |
| 2017 holdout RMSE | 34.13 µg/m³ |
| 2017 holdout R² | 0.912 |

### Understanding the metrics

- **MAE (Mean Absolute Error):** The average absolute difference between predicted and observed values.
- **RMSE (Root Mean Squared Error):** Measures prediction error while giving larger errors more influence.
- **R² (Coefficient of Determination):** Describes how well predictions explain variation in the target values relative to a baseline.

### Evaluation caveat

These results describe performance on the historical project dataset. They **must not be interpreted as measured accuracy on current live 2026 data**.

The deployed application obtains recent prediction inputs from Open-Meteo, while the historical model was trained and evaluated on the project's historical dataset. Differences between these sources may affect live performance. Further validation on appropriately matched observations is necessary before operational reliance.

## Application Workflow

1. The user opens the AirIQ dashboard.
2. The frontend requests the supported station list from the backend.
3. The user selects a monitoring station.
4. The user clicks **Generate Next-Hour Forecast**.
5. The frontend sends the selected station to the `/predict` endpoint.
6. The backend validates the station.
7. The backend retrieves recent weather and air-quality observations.
8. The backend combines the observations by timestamp.
9. The required temporal, lag, and rolling features are constructed.
10. The saved Random Forest pipeline generates the next-hour PM2.5 estimate.
11. The backend returns the prediction and relevant timestamps.
12. The dashboard displays the result and supporting environmental context.

## API Endpoints

| HTTP method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Confirms that the prediction API is running |
| `GET` | `/stations` | Returns the supported Beijing monitoring stations |
| `POST` | `/predict` | Accepts a station name and returns a next-hour prediction |

Example request body for `POST /predict`:

```json
{
  "station": "Aotizhongxin"
}
```

The backend checks the station name before generating a prediction. The precise response schema should be confirmed using the running application.

### Interactive API documentation

When the backend is running locally, FastAPI's interactive documentation is available at:

`http://127.0.0.1:8000/docs`

## Installation and Setup

> **Note:** Confirm the actual folder names, dependency files, model-artifact paths, and frontend configuration in your repository before running these commands. The commands below are a starting point based on the documented application architecture.

### Prerequisites

- Python
- Node.js and npm
- AirIQ source code
- Backend dependencies
- Frontend dependencies
- The trained Random Forest Joblib model artifact
- Internet access for Open-Meteo requests

### 1. Start the backend

Open a terminal at the project root—the directory containing the `backend` folder.

Create and activate a virtual environment if appropriate for your setup:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install dependencies using the dependency file included in your repository. For example, if the project contains `backend\requirements.txt`:

```powershell
pip install -r backend\requirements.txt
```

If that file does not exist, use the actual dependency file supplied with the project.

Start the FastAPI application from the project root:

```powershell
py -m uvicorn backend.main:app --reload
```

If startup succeeds, test the following endpoints:

- API health: `http://127.0.0.1:8000/`
- Supported stations: `http://127.0.0.1:8000/stations`
- API documentation: `http://127.0.0.1:8000/docs`

The backend must be able to locate the trained model artifact and any required configuration files.

### 2. Start the frontend

Open a separate terminal and navigate to the frontend directory used by your repository.

If the directory is named `frontend`, run:

```powershell
cd frontend
npm install
npm run dev
```

Use the actual frontend folder name if it differs. Open the local URL printed by Vite in the terminal.

If the frontend needs a backend base URL, configure it according to the environment variables or configuration already used in your project.

### 3. Generate a forecast

1. Open the frontend dashboard.
2. Select one of the supported Beijing stations.
3. Click **Generate Next-Hour Forecast**.
4. Review the predicted PM2.5 value, predicted time, unit, and supporting information.

## Testing and Monitoring

The project system description identifies three testing levels.

### Backend testing

- API startup
- Station retrieval
- Input validation
- Model loading
- Prediction requests

### Frontend testing

- Station selection
- Forecast generation
- Loading and error states
- Result display
- Reset behaviour

### End-to-end testing

Verify that selecting a station in the frontend sends a request to FastAPI, triggers live-data retrieval and model inference, and returns a prediction to the dashboard.

FastAPI request logs can help identify successful requests and API errors. This is technical system monitoring rather than personal user tracking.

## Limitations

- Historical evaluation results do not establish prediction accuracy on current live data.
- Historical training and live prediction inputs come from different sources and may have different distributions.
- The current model is trained using Beijing data; performance in other cities or countries has not been established.
- Performance during extreme or unusual pollution events requires further evaluation.
- Reliability of third-party data feeds and long-term production performance require additional monitoring.
- Forecasts should support human judgement rather than replace official environmental guidance or professional decisions.

## Future Improvements

Potential future enhancements include:

- Retraining with newer, appropriately validated observations
- Evaluating performance against matched live observations
- Monitoring prediction errors and data-source reliability
- Improving error handling when external services are unavailable
- Adding historical forecast views and trend comparisons
- Validating the approach with suitable data from other locations
- Preparing the application for secure and reliable public deployment

## Data Sources and Acknowledgements

- **Historical dataset:** Beijing Multi-Site Air-Quality dataset, UCI Machine Learning Repository.
- **Live environmental information:** Open-Meteo Weather API and Open-Meteo Air Quality API.

Please check the providers' current terms and attribution requirements before redistributing or deploying the application.

## Disclaimer

AirIQ is an academic prototype developed for a data mining mini project. Forecasts are estimates and should not be treated as official air-quality alerts or as a substitute for validated public-health or environmental guidance.

---

**Project:** AirIQ — Next-Hour PM2.5 Air Quality Forecasting System  
**Module:** IT3051 – Fundamentals of Data Mining  
**Institution:** SLIIT  
**Year:** 2026
