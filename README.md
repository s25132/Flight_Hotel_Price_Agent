# Flight_Hotel_Price_Agent

Architektura komponentów
1) Dane

MinIO (S3) trzyma surowe snapshoty:

s3://flight-data/raw/2026-03-02/flights.csv

s3://flight-data/raw/2026-03-09/flights.csv

opcjonalnie: processed/, features/ też możesz trzymać w MinIO

2) Trening + rejestracja

Airflow (DAG):

pobierz raw z MinIO

waliduj schemat (opcjonalnie)

feature engineering + split

train

eval

compute SHAP background + global importance

log do MLflow

register model w MLflow Registry

promote do Production (warunkowo)

3) Serving

FastAPI “model-service”:

POST /predict – predykcja ceny

POST /explain – SHAP dla pojedynczego rekordu (top feature contributions)

GET /health

FastAPI ładuje model z:

models:/flight_price_model/Production (MLflow)

4) MCP + Agent LLM

MCP server udostępnia narzędzia:

tool predict → wewnętrznie woła model-service /predict

tool explain → wewnętrznie woła model-service /explain

Agent LLM (Twoje /assist albo osobny “agent-service”) ma podpięte MCP tools i generuje odpowiedź czatową:

“dlaczego taka cena” → na podstawie SHAP

“co sprawdzić” → checklista oparta o top cechy

5) Frontend do czatu

Najprościej: Streamlit Chat UI:

użytkownik wpisuje parametry lotu + pytanie

UI wysyła do agent endpointu /chat

agent używa MCP tools i odpowiada