# Quick Testing Guide for Exoplanet ML API Integration

## 🚀 Test the Integration

### Step 1: Access the Explorer
URL: `https://victorious-smoke-0c422eb10.2.azurestaticapps.net/explorer`

### Step 2: Load Test Data
Click the **"🎲 Load Dummy Data"** button to auto-fill all fields with:

| Parameter | Value | Description |
|-----------|-------|-------------|
| period | 3.52 | Orbital period in days |
| duration | 2.1 | Transit duration in hours |
| depth | 500.0 | Transit depth in ppm |
| radius | 1.2 | Planet radius in Earth radii |
| eqt | 800.0 | Equilibrium temperature in K |
| insol | 15.0 | Insolation flux |
| st_teff | 5800.0 | Stellar temperature in K |
| st_logg | 4.5 | Stellar surface gravity |
| st_rad | 1.0 | Stellar radius in solar radii |
| ra | 123.456 | Right ascension in degrees |
| dec | -12.345 | Declination in degrees |
| source_id | TIC-12345 | Source identifier |
| mission_code | TESS | Mission name |

### Step 3: Run Prediction
Click **"⚡ Run Prediction"** button

### Step 4: Expected Results
Based on the curl example provided, you should see:

**Classification:**
- Prediction: **CANDIDATE**
- Confidence: **45.2%**

**Probability Breakdown:**
- Candidate: **45.2%**
- False Positive: **18.6%**
- Planet: **36.2%**

**SHAP Values:**
- Period: 0.0424
- Duration: -0.1159

**TabNet Importance:**
(Full importance scores displayed)

### Step 5: Test Chat Interface
After getting results, try these questions:

#### 💬 Sample Questions:
1. **"What's the confidence level?"**
   - Will explain the 45.2% confidence and probability breakdown

2. **"Explain the SHAP values"**
   - Will interpret feature importance scores

3. **"Is this a false positive?"**
   - Will discuss the 18.6% false positive probability

4. **"What are the next steps?"**
   - Will provide follow-up recommendations for candidates

5. **"Tell me about the temperature"**
   - Will explain 800K equilibrium temperature and habitability

6. **"How big is this planet?"**
   - Will explain 1.2 Earth radii and Super-Earth classification

## 🔧 Troubleshooting

### Issue: API Call Fails
**Check:**
- Open browser DevTools (F12)
- Go to Console tab
- Look for error messages
- Check Network tab for failed requests

**Common Causes:**
- CORS not enabled on API
- API endpoint temporarily down
- Missing/invalid input fields
- Network connectivity issues

### Issue: No Results Displayed
**Check:**
- Ensure all 13 fields are filled
- Check for JavaScript errors in console
- Verify API response format matches expected structure

### Issue: Chat Not Working
**Note:** Chat only enables after a successful prediction
- Run prediction first
- Then try asking questions

## 🧪 Custom Test Cases

### Test Case 1: Hot Jupiter
```json
{
  "period": "3.5",
  "duration": "3.0",
  "depth": "10000",
  "radius": "10.0",
  "eqt": "1500",
  "insol": "1000",
  "st_teff": "6000",
  "st_logg": "4.4",
  "st_rad": "1.2",
  "ra": "180.0",
  "dec": "45.0",
  "source_id": "TIC-99999",
  "mission_code": "TESS"
}
```
Expected: Likely classified as Gas Giant

### Test Case 2: Earth-like
```json
{
  "period": "365.25",
  "duration": "13.0",
  "depth": "84",
  "radius": "1.0",
  "eqt": "288",
  "insol": "1.0",
  "st_teff": "5778",
  "st_logg": "4.44",
  "st_rad": "1.0",
  "ra": "200.0",
  "dec": "-30.0",
  "source_id": "TIC-EARTH",
  "mission_code": "TESS"
}
```
Expected: Terrestrial planet in habitable zone

### Test Case 3: Super-Earth
```json
{
  "period": "10.0",
  "duration": "4.5",
  "depth": "300",
  "radius": "1.8",
  "eqt": "400",
  "insol": "5.0",
  "st_teff": "5500",
  "st_logg": "4.5",
  "st_rad": "0.9",
  "ra": "150.0",
  "dec": "20.0",
  "source_id": "TIC-55555",
  "mission_code": "Kepler"
}
```
Expected: Super-Earth classification

## 📊 What to Check After Integration

### ✅ Checklist:
- [ ] API endpoint is reachable
- [ ] All 13 input fields are visible
- [ ] "Load Dummy Data" button works
- [ ] "Clear All" button works
- [ ] Form validation prevents empty submissions
- [ ] "Run Prediction" button shows loading state
- [ ] API response is received and logged in console
- [ ] Results are displayed in LLM response section
- [ ] Quick stats cards show correct values
- [ ] Chat interface becomes enabled
- [ ] Chat responses are contextual and helpful
- [ ] No JavaScript errors in console
- [ ] UI is responsive on mobile/desktop

## 🎯 Next Steps: AI Agent Integration

Once you verify the ML model integration works, you can add an AI agent layer:

```javascript
// After getting modelPrediction, send to AI agent
const aiAgentRequest = {
  prediction: modelPrediction,
  inputParameters: modelInputs,
  userRequest: "Provide follow-up analysis and recommendations"
};

const aiAgentResponse = await fetch('YOUR_AI_AGENT_ENDPOINT', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(aiAgentRequest)
});

const agentGuidance = await aiAgentResponse.json();
// Display agent guidance to user
```

This will allow the AI agent to:
1. Receive the ML model prediction
2. Analyze the results
3. Provide expert recommendations
4. Guide next steps
5. Answer follow-up questions

## 📝 Documentation
See `API_INTEGRATION_SUMMARY.md` for complete technical details.
