# Exoplanet Detection System - Data Flow Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                               │
│              (Azure Static Web Apps - Frontend)                      │
│          https://victorious-smoke-0c422eb10.2.azurestaticapps.net   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ User fills 13 input fields
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Light Curve Explorer Component                    │
│                  /src/pages/LightCurveExplorer.js                   │
├─────────────────────────────────────────────────────────────────────┤
│  Input Fields (13):                                                  │
│  ✓ period, duration, depth, radius, eqt, insol                      │
│  ✓ st_teff, st_logg, st_rad                                         │
│  ✓ ra, dec, source_id, mission_code                                 │
│                                                                       │
│  Actions:                                                            │
│  • Load Dummy Data                                                   │
│  • Clear All Fields                                                  │
│  • Upload CSV                                                        │
│  • Run Prediction ────────────────────────┐                         │
└───────────────────────────────────────────┼─────────────────────────┘
                                             │
                                             │ HTTP POST Request
                                             │ Content-Type: application/json
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ML MODEL API ENDPOINT                           │
│              (Azure Container Apps - Python/FastAPI)                 │
│   https://exoplanet-api.agreeableocean-4e4135ec.eastus             │
│                .azurecontainerapps.io/predict                        │
├─────────────────────────────────────────────────────────────────────┤
│  Request Payload:                                                    │
│  {                                                                   │
│    "period": 3.52,                                                   │
│    "duration": 2.1,                                                  │
│    "depth": 500.0,                                                   │
│    "radius": 1.2,                                                    │
│    "eqt": 800.0,                                                     │
│    "insol": 15.0,                                                    │
│    "st_teff": 5800.0,                                                │
│    "st_logg": 4.5,                                                   │
│    "st_rad": 1.0,                                                    │
│    "ra": 123.456,                                                    │
│    "dec": -12.345,                                                   │
│    "source_id": "TIC-12345",                                         │
│    "mission_code": "TESS"                                            │
│  }                                                                   │
└─────────────────────────────────────────────────────────────────────┘
                                             │
                                             │ ML Processing
                                             │ • CatBoost Model
                                             │ • TabNet Model
                                             │ • Ensemble Prediction
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       API RESPONSE                                   │
├─────────────────────────────────────────────────────────────────────┤
│  {                                                                   │
│    "prediction": "candidate",                                        │
│    "confidence": 0.452,                                              │
│    "probabilities": {                                                │
│      "candidate": 0.452,                                             │
│      "false_positive": 0.186,                                        │
│      "planet": 0.362                                                 │
│    },                                                                │
│    "catboost_shap": {                                                │
│      "period": 0.0424,                                               │
│      "duration": -0.1159                                             │
│    },                                                                │
│    "tabnet_importance": {                                            │
│      "period": 0.0,                                                  │
│      "duration": 0.0574,                                             │
│      "depth": 0.0576,                                                │
│      ...                                                             │
│    }                                                                 │
│  }                                                                   │
└─────────────────────────────────────────────────────────────────────┘
                                             │
                                             │ Parse & Transform
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    RESULTS DISPLAY SECTION                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │           📝 LLM RESPONSE (Generated)                        │   │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │  ⚠️🔍 Exoplanet Detection Analysis Complete!                │   │
│  │                                                               │   │
│  │  🎯 Classification Result: CANDIDATE                         │   │
│  │  The AI ensemble model has analyzed your transit signal      │   │
│  │  with 45.2% confidence.                                      │   │
│  │                                                               │   │
│  │  📊 Probability Breakdown:                                   │   │
│  │  • Planet: 36.2%                                             │   │
│  │  • Candidate: 45.2%                                          │   │
│  │  • False Positive: 18.6%                                     │   │
│  │                                                               │   │
│  │  🔬 Model Insights (SHAP):                                   │   │
│  │  • Period Impact: 0.0424 (increases planet likelihood)       │   │
│  │  • Duration Impact: -0.1159 (decreases planet likelihood)    │   │
│  │                                                               │   │
│  │  🔍 Recommendation: Additional observations warranted...     │   │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              📊 QUICK STATS CARDS (9 cards)                  │   │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │  [CANDIDATE] [45.2%] [36.2%] [45.2%] [18.6%]                │   │
│  │  [Super-Earth] [1.2 R⊕] [800 K] [3.52 days]                 │   │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │           💬 CHAT INTERFACE                                   │   │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │  User: "What's the confidence level?"                        │   │
│  │  Bot: "The model has 45.2% confidence in this                │   │
│  │       classification as 'candidate'..."                      │   │
│  │                                                               │   │
│  │  [Type your question here...] [Send →]                       │   │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                             │
                                             │ (Future Enhancement)
                                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  🤖 AI AGENT LAYER (Optional)                        │
├─────────────────────────────────────────────────────────────────────┤
│  • Receives ML model prediction                                      │
│  • Analyzes results in context                                       │
│  • Provides expert recommendations                                   │
│  • Guides follow-up observations                                     │
│  • Answers domain-specific questions                                 │
└─────────────────────────────────────────────────────────────────────┘
```

## Component Interaction Flow

### 1. User Input Phase
```
User fills form → Validation → Click "Run Prediction"
```

### 2. API Request Phase
```
Convert inputs to numbers → Build JSON payload → POST to API
```

### 3. ML Processing Phase (Server-side)
```
Receive request → Run CatBoost → Run TabNet → Ensemble → Return prediction
```

### 4. Response Processing Phase
```
Parse API response → Transform to display format → Generate LLM text
```

### 5. Display Phase
```
Show LLM response → Render stats cards → Enable chat interface
```

### 6. Chat Interaction Phase
```
User asks question → Context-aware response → Display in chat
```

## Key Data Transformations

### Input → API
```javascript
// Frontend State (Strings)
{
  period: "3.52",
  duration: "2.1",
  ...
}

// Transformed to API Payload (Numbers)
{
  period: 3.52,
  duration: 2.1,
  ...
}
```

### API → Display
```javascript
// API Response
{
  prediction: "candidate",
  confidence: 0.452,
  probabilities: { candidate: 0.452, ... }
}

// Transformed for Display
{
  prediction: "candidate",
  confidence: "45.2",  // percentage
  candidateProb: "45.2",
  planetType: "Super-Earth",  // calculated
  ...
}
```

## Error Handling

```
Input Validation
    ↓ (fail)
    └─→ Alert user about missing/invalid fields

API Call
    ↓ (fail)
    └─→ Catch error → Log to console → Alert user

Response Parsing
    ↓ (fail)
    └─→ Handle gracefully → Show default message
```

## Security Considerations

✅ **Current Implementation:**
- Client-side validation
- No authentication required (public API)
- HTTPS endpoints only
- No sensitive data storage

⚠️ **Recommendations:**
- Add rate limiting on frontend
- Implement API key if needed
- Add request timeout handling
- Consider caching for repeated queries

## Performance Optimization

🚀 **Current:**
- Direct API calls
- Real-time processing
- No caching

💡 **Future Enhancements:**
- Cache recent predictions (localStorage)
- Debounce rapid submissions
- Batch processing for multiple targets
- WebSocket for long-running tasks

## Monitoring & Analytics

📊 **Track:**
- API call success/failure rate
- Average response time
- User interaction patterns
- Most common input patterns
- Chat question topics

## Deployment Status

| Component | Status | URL |
|-----------|--------|-----|
| Frontend | ✅ Deployed | Azure Static Web Apps |
| ML API | ✅ Deployed | Azure Container Apps |
| Integration | ✅ Complete | Connected & Working |

## Version History

- **v1.0** - Initial mock implementation
- **v2.0** - Real API integration (Current)
- **v2.1** - AI agent integration (Planned)
