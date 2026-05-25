# Fase 5 — IA y Optimización (Año 2 — 2027)

## Objetivo

Aplicar inteligencia artificial para optimizar las operaciones del taller: predicción de demanda de repuestos, detección automática de anomalías financieras y asistente de diagnóstico para mecánicos.

## Prerequisitos

- Al menos 12 meses de datos históricos en el sistema
- Flujos de negocio estabilizados y maduros
- Equipo técnico capaz de mantener modelos ML en producción
- Presupuesto cloud adicional para cómputo ML (~$30-50 USD/mes)

## Módulos de IA

### Predicción de Demanda de Repuestos

```python
# Modelo: Facebook Prophet para series temporales
# Datos: historial de movimientos de inventario (12 meses)
# Output: cantidad óptima de reposición para los próximos 30 días

from prophet import Prophet

def forecast_part_demand(part_id: str, days_ahead: int = 30):
    df = get_inventory_movements(part_id)  # ds, y (cantidad)
    
    m = Prophet(
        seasonality_mode='multiplicative',
        weekly_seasonality=True,   # Lunes vs sábado difieren
        yearly_seasonality=False,  # Solo 12 meses de data
    )
    m.fit(df)
    
    future = m.make_future_dataframe(periods=days_ahead)
    forecast = m.predict(future)
    
    # Si forecast > stock_actual → generar orden de compra sugerida
    return forecast[['ds', 'yhat', 'yhat_lower']].tail(days_ahead)
```

**Valor:** Reducir stockouts de repuestos críticos en 80%.

### Detección de Anomalías Financieras

```python
# Modelo: Isolation Forest (sin supervisión)
# Datos: transacciones financieras con features temporales
# Output: transacciones estadísticamente anómalas para revisión

from sklearn.ensemble import IsolationForest

def detect_financial_anomalies(transactions_df):
    features = [
        'amount',              # Monto de la transacción
        'hour_of_day',         # Hora del día (fuera de horario = sospechoso)
        'day_of_week',         # Domingo = sospechoso
        'days_since_last_tx',  # Gap inusual entre transacciones
        'amount_vs_avg_30d',   # Diferencia vs promedio últimos 30 días
    ]
    
    clf = IsolationForest(contamination=0.05, random_state=42)
    clf.fit(transactions_df[features])
    
    predictions = clf.predict(transactions_df[features])
    return transactions_df[predictions == -1]  # -1 = anomalía
```

**Valor:** Detección proactiva de patrones de fraude antes de revisión manual.

### Asistente de Diagnóstico para Mecánicos

```typescript
// Integración con Claude Haiku para sugerencias
async function getSuggestedDiagnosis(
  symptoms: string[],
  vehicleModel: string,
  year: number,
): Promise<DiagnosisSuggestion[]> {
  const response = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 500,
    system: `Eres un experto mecánico automotriz. 
    Sugiere diagnósticos posibles y repuestos típicos para vehículo ${vehicleModel} ${year}.
    Responde en español. Formato JSON.`,
    messages: [{
      role: 'user',
      content: `Síntomas reportados: ${symptoms.join(', ')}`,
    }],
  })
  
  return parseDiagnosisResponse(response.content[0].text)
  // Costo: ~$0.001 por consulta
}
```

**Valor:** Diagnósticos más rápidos y precisos, reducir tiempo promedio de OT.

### OCR de Placas (Tesseract)

```typescript
// Lectura automática de placa con cámara de tablet
import Tesseract from 'tesseract.js'

async function readPlateFromCamera(imageData: Blob): Promise<string> {
  const { data: { text } } = await Tesseract.recognize(imageData, 'eng', {
    // Configuración para placas peruanas: ABC-123 format
    tessedit_char_whitelist: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-',
  })
  
  const platePattern = /[A-Z]{3}-\d{3}|[A-Z]{2}\d{4}/
  const match = text.match(platePattern)
  return match ? match[0] : null
}
```

**Valor:** Eliminar errores de digitación manual de placas (ya implementado en Fase 2+).

## Costos Estimados de IA (Mensuales)

| Servicio | Uso estimado | Costo |
|---------|-------------|-------|
| Claude Haiku (diagnóstico) | 100 consultas/mes | ~$0.50 |
| GPT-4o-mini (clasificación OTs) | 200 OTs/mes | ~$0.02 |
| Prophet (demand forecasting) | Self-hosted en Railway | $0 |
| Isolation Forest (anomalías) | Self-hosted en Railway | $0 |
| **Total IA mensual** | | **~$5 USD** |

## Migración a AWS (Fase 5)

Con el volumen de datos de 12+ meses, la migración de Railway → AWS se vuelve conveniente:

```
Railway (MVP/Prod año 1)     →     AWS (año 2+)
  PostgreSQL (Railway)       →     RDS PostgreSQL 15 (Multi-AZ)
  Redis (Railway)            →     ElastiCache Redis
  NestJS (Railway)           →     ECS Fargate + ALB
  Archivos (Supabase)        →     S3 + CloudFront
  Auth (Supabase)            →     JWT RS256 custom
  
Costo: ~$150-300 USD/mes (vs $65-120 MVP)
Ganancia: Auto-scaling, Multi-AZ, SLAs enterprise
```
