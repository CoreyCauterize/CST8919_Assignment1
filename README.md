# CST8919 Lab 1: Implementing User Login with Flask and Auth0
Corey Mark-Stewart

[Video](https://youtu.be/0Jw4v3ZA_7o)

## Instructions
### Set up yout enviroment
Create a virtual environment
```sh
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```
### Install dependencies
```sh
pip install -r requirements.txt
```
### Environment Variables example
.env example 
```ini
# Auth0 Configuration
AUTH0_DOMAIN=YOUR_AUTH0_DOMAIN
AUTH0_CLIENT_ID=YOUR_CLIENT_ID
AUTH0_CLIENT_SECRET=YOUR_CLIENT_SECRET
AUTH0_SECRET=YOUR_GENERATED_SECRET
AUTH0_REDIRECT_URI=http://localhost:5000/callback
```

### Run app locally

```sh
python app.py
```
