# Solar Quote API - Render Deployment Guide

This document explains how the Solar Quote API is hosted on [Render](https://render.com).

## Render Service Overview

This project is hosted using Render's free tier instance. Render's free instances have the following characteristics:

- Automatically enters sleep mode after 15 minutes of inactivity
- Restarts upon the first request, which may cause "cold start" delays
- Limited runtime hours per month
- Lower performance specifications

## Application Functionality

The `app.js` file contains the core functionality of the Solar Quote API:

1. **Quote Request Handling**: Processes solar quote requests through the `/api/send-quote-request` endpoint
   - Accepts form submissions with customer information
   - Handles file uploads (such as roof images)
   - Processes different calculator modes (standard, advanced, assistive)

2. **Email Service**: Uses Nodemailer to send formatted quote requests
   - Customized email templates for different calculator modes
   - Attachments support for uploaded files
   - Configurable recipient and CC addresses

3. **CORS Configuration**: Implements secure Cross-Origin Resource Sharing
   - Restricts access to specified frontend origins
   - Handles preflight requests
   - Configurable through environment variables

4. **Health Monitoring**: Provides health check endpoints
   - Basic `/` route for simple status checks
   - Detailed `/health` endpoint with timestamp information

5. **Sleep Prevention**: Implements a `keepAlive` function to prevent Render's free tier from sleeping
   - Self-pinging mechanism to maintain activity
   - Configurable through environment variables

## Environment Variables Setup

The following environment variables need to be configured in the Render dashboard:

| Environment Variable | Description | Example |
|----------|------|------|
| `EMAIL_PASSWORD` | App-specific password for the Gmail account | `************` |
| `EMAIL_USER` | Gmail account used for sending emails | `************` |
| `FRONTEND_URL` | Frontend URLs allowed by CORS (comma-separated) | `************` |
| `KEEP_ALIVE` | Enable prevention of Render instance sleep mode | `************` |
| `NODE_ENV` | Node.js environment setting | `************` |
| `PORT` | Port for the server to run on | `************` |
| `RECIPIENT_EMAIL` | Email address to receive quote requests | `************` |
| `SERVER_URL` | Server URL (for the keepAlive function) | `************` |

## Setting Up a Service on Render

1. Log in to the [Render dashboard](https://dashboard.render.com/).
2. Click the `New +` button and select `Web Service`.
3. Connect to this repository or provide a public repository URL.
4. Configure the following settings:
   - **Name**: `solar-calculator-backend` (or your preferred name)
   - **Environment**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `node app.js`
   - **Plan**: Free

5. Add the environment variables listed above in the `Environment Variables` section.
6. Click the `Create Web Service` button.

## Sleep Prevention Feature

Free Render instances automatically go into sleep mode after 15 minutes of inactivity. To prevent this, the app includes a `keepAlive` function that sends a self-ping to the server every 14 minutes to keep it active.

To use this feature:
1. Set the `KEEP_ALIVE` environment variable to `true`.
2. Set the `SERVER_URL` environment variable to the URL of your deployed service.

```javascript
// keepAlive function in app.js
function keepAlive() {
  const url = process.env.SERVER_URL || 'https://solar-calculator-backend.onrender.com';
  
  // Send ping every 14 minutes (slightly shorter than Render's 15-minute timeout)
  setInterval(() => {
    fetch(url)
      .then(response => {
        console.log(`Keep-alive ping sent at ${new Date().toISOString()}, status: ${response.status}`);
      })
      .catch(error => {
        console.error(`Keep-alive ping failed: ${error.message}`);
      });
  }, 14 * 60 * 1000); // 14 minutes (in milliseconds)
}
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Basic health check and API status |
| `/health` | GET | Detailed health status with timestamp |
| `/api/send-quote-request` | POST | Process quote request forms with optional file attachments |

## Viewing Logs

You can view application logs through the `Logs` tab in the Render dashboard, which is useful for troubleshooting.

## Known Limitations

1. Free instances are limited to 750 hours per month (sufficient for a single service).
2. There may be slight delays during cold starts.
3. CPU and memory resources are limited.

## Migrating to Production Environment

For actual production environments, consider migrating to Render's paid plans or other cloud services (AWS, GCP, Azure, etc.). 