# Deployment Instructions

## 🚀 Deploy Updated Frontend to Azure Static Web Apps

### Option 1: Automatic Deployment (Recommended)

If your Azure Static Web App is connected to GitHub:

```bash
# 1. Commit the changes
git add src/pages/LightCurveExplorer.js
git commit -m "Integrate ML model API for exoplanet detection"

# 2. Push to repository
git push origin main

# 3. Azure will automatically build and deploy
# Monitor deployment at: https://portal.azure.com
```

### Option 2: Manual Build & Deploy

```bash
# 1. Navigate to frontend directory
cd /home/hamza/hamzaa/personal-things/NASA-HACKATHON/spaceapps/frontend

# 2. Install dependencies (if needed)
npm install --legacy-peer-deps

# 3. Build the production version
npm run build

# 4. Test locally (optional)
npx serve -s build

# 5. Deploy to Azure Static Web Apps
# Using Azure CLI:
az staticwebapp deploy \
  --name victorious-smoke-0c422eb10 \
  --resource-group <your-resource-group> \
  --app-location ./build

# Or manually upload the build folder via Azure Portal
```

### Option 3: Quick Deploy with SWA CLI

```bash
# 1. Install Azure Static Web Apps CLI
npm install -g @azure/static-web-apps-cli

# 2. Build the app
npm run build

# 3. Deploy
swa deploy ./build \
  --deployment-token <your-deployment-token> \
  --env production
```

## 📋 Pre-Deployment Checklist

Before deploying, verify:

- [ ] All changes are committed to git
- [ ] No JavaScript errors in console
- [ ] Build completes successfully (`npm run build`)
- [ ] All tests pass (`npm test` - if you have tests)
- [ ] Tested locally with `npm start`
- [ ] API endpoint is accessible from browser
- [ ] CORS is enabled on API endpoint

## 🔍 Testing the Deployment

After deployment completes:

1. **Clear Browser Cache**
   ```
   Chrome/Edge: Ctrl+Shift+Delete
   Firefox: Ctrl+Shift+Delete
   Safari: Cmd+Option+E
   ```

2. **Visit the Explorer Page**
   ```
   https://victorious-smoke-0c422eb10.2.azurestaticapps.net/explorer
   ```

3. **Open Browser DevTools** (F12)
   - Check Console tab for errors
   - Check Network tab for API calls

4. **Test the Integration**
   - Click "Load Dummy Data"
   - Click "Run Prediction"
   - Verify API call in Network tab
   - Check response in Console
   - Verify results display correctly

## 🐛 Troubleshooting

### Issue: Changes Not Appearing

**Solution 1: Clear Cache**
```bash
# Hard refresh in browser
Ctrl+F5 (Windows/Linux)
Cmd+Shift+R (Mac)
```

**Solution 2: Check Deployment Status**
```bash
# View deployment logs in Azure Portal
# Navigate to: Static Web App → Deployments
```

**Solution 3: Verify Build**
```bash
# Check if build/ directory is updated
ls -la build/
cat build/asset-manifest.json
```

### Issue: API Calls Failing

**Check 1: CORS Configuration**
The API must allow requests from your frontend domain:
```python
# On the API side (FastAPI example)
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://victorious-smoke-0c422eb10.2.azurestaticapps.net",
        "http://localhost:3000"  # for development
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Check 2: API Endpoint**
Verify the API is accessible:
```bash
curl -X POST https://exoplanet-api.agreeableocean-4e4135ec.eastus.azurecontainerapps.io/predict \
  -H "Content-Type: application/json" \
  -d '{"period": 3.52, "duration": 2.1, "depth": 500.0, "radius": 1.2, "eqt": 800.0, "insol": 15.0, "st_teff": 5800.0, "st_logg": 4.5, "st_rad": 1.0, "ra": 123.456, "dec": -12.345, "source_id": "TIC-12345", "mission_code": "TESS"}'
```

**Check 3: Network Tab**
In browser DevTools → Network tab:
- Look for the `/predict` request
- Check request payload
- Check response status (should be 200)
- View response preview

### Issue: Build Failures

**Error: Dependencies**
```bash
# Clean install
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

**Error: Memory**
```bash
# Increase Node.js memory
export NODE_OPTIONS=--max-old-space-size=4096
npm run build
```

**Error: ESLint Warnings**
```bash
# Build with CI=false (already configured)
CI=false npm run build
```

## 🔐 Environment Variables

Currently, the API endpoint is hardcoded. To use environment variables:

### 1. Create `.env` file (for local development)
```bash
REACT_APP_EXOPLANET_API_URL=https://exoplanet-api.agreeableocean-4e4135ec.eastus.azurecontainerapps.io
```

### 2. Update Code
```javascript
// In LightCurveExplorer.js
const API_URL = process.env.REACT_APP_EXOPLANET_API_URL || 
                'https://exoplanet-api.agreeableocean-4e4135ec.eastus.azurecontainerapps.io';

const response = await fetch(`${API_URL}/predict`, {
  // ...
});
```

### 3. Configure in Azure Static Web Apps
```bash
# Via Azure CLI
az staticwebapp appsettings set \
  --name victorious-smoke-0c422eb10 \
  --setting-names REACT_APP_EXOPLANET_API_URL=https://exoplanet-api.agreeableocean-4e4135ec.eastus.azurecontainerapps.io
```

Or via Azure Portal:
1. Go to Static Web App resource
2. Navigate to Configuration
3. Add Application Setting
4. Key: `REACT_APP_EXOPLANET_API_URL`
5. Value: `https://exoplanet-api.agreeableocean-4e4135ec.eastus.azurecontainerapps.io`

## 📊 Monitoring After Deployment

### Azure Portal
1. Navigate to your Static Web App
2. Check "Overview" for health status
3. Check "Metrics" for usage stats
4. Check "Log Stream" for real-time logs

### Browser Console
Monitor for:
- API request/response logs
- Any JavaScript errors
- Performance warnings

### API Monitoring
Check your Azure Container App logs:
```bash
az containerapp logs show \
  --name exoplanet-api \
  --resource-group <your-resource-group> \
  --follow
```

## 🎯 Post-Deployment Verification

Run through this checklist:

1. **Homepage Loads** ✓
   - Navigate to root URL
   - All components render

2. **Explorer Page Loads** ✓
   - Navigate to `/explorer`
   - All 13 input fields visible

3. **Load Dummy Data** ✓
   - Click button
   - All fields populate

4. **Run Prediction** ✓
   - Click Run Prediction
   - Loading spinner appears
   - No console errors

5. **API Call Success** ✓
   - Check Network tab
   - Status 200
   - Response received

6. **Results Display** ✓
   - LLM response appears
   - Stats cards populate
   - Confidence percentages show

7. **Chat Interface** ✓
   - Chat becomes enabled
   - Type a question
   - Receive response

8. **Mobile Responsive** ✓
   - Test on mobile device
   - UI adapts properly

## 🔄 Rollback Procedure

If issues occur after deployment:

### Via GitHub (if auto-deploy)
```bash
git revert HEAD
git push origin main
```

### Via Azure Portal
1. Navigate to Static Web App
2. Go to "Deployments"
3. Find previous successful deployment
4. Click "Reactivate"

### Manual Rollback
```bash
# Checkout previous version
git checkout <previous-commit-hash>

# Build and deploy
npm run build
az staticwebapp deploy --name victorious-smoke-0c422eb10 --app-location ./build
```

## 📞 Support

If you encounter issues:

1. **Check Logs**
   - Browser console
   - Azure deployment logs
   - API container logs

2. **Verify Configuration**
   - API endpoint URL
   - CORS settings
   - Environment variables

3. **Test Components**
   - Test API directly with curl
   - Test frontend locally
   - Test after clearing cache

4. **Documentation**
   - See `API_INTEGRATION_SUMMARY.md`
   - See `TESTING_GUIDE.md`
   - See `ARCHITECTURE.md`

## ✅ Success Criteria

Deployment is successful when:

✓ Build completes without errors  
✓ Deployment to Azure succeeds  
✓ Explorer page loads correctly  
✓ All 13 input fields are visible  
✓ Dummy data button works  
✓ API calls return valid responses  
✓ Results display correctly  
✓ Chat interface is functional  
✓ No JavaScript console errors  
✓ Mobile responsive design works  

## 🚀 Ready to Deploy!

Your integration is complete and ready for deployment. The changes have been made to:

- ✅ `/src/pages/LightCurveExplorer.js` - Updated with full API integration

Simply commit, push, and let Azure handle the rest!

```bash
git add .
git commit -m "Integrate ML model API with Light Curve Explorer"
git push origin main
```

Then test at: `https://victorious-smoke-0c422eb10.2.azurestaticapps.net/explorer`
