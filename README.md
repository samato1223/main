# Python3 Flask backend for Copywriting Finder App
# Includes Yelp Fusion API integration and OpenAI rewrite

from flask import Flask, request, jsonify
from flask_cors import CORS
import requests
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

YELP_API_KEY = os.getenv("YELP_API_KEY")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

@app.route("/api/businesses", methods=["GET"])
def search_businesses():
    location = request.args.get("location", default="San Francisco")
    term = request.args.get("term", default="small business")

    try:
        headers = {"Authorization": f"Bearer {YELP_API_KEY}"}
        params = {
            "term": term,
            "location": location,
            "limit": 5
        }
        response = requests.get("https://api.yelp.com/v3/businesses/search", headers=headers, params=params)
        response.raise_for_status()
        return jsonify(response.json().get("businesses", []))
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route("/api/rewrite", methods=["POST"])
def rewrite_copy():
    data = request.get_json()
    text = data.get("text")

    try:
        headers = {
            "Authorization": f"Bearer {OPENAI_API_KEY}",
            "Content-Type": "application/json"
        }
        payload = {
            "model": "gpt-4",
            "messages": [
                {"role": "system", "content": "You are a professional copywriter."},
                {"role": "user", "content": f"Rewrite this for maximum marketing impact: {text}"}
            ]
        }
        response = requests.post("https://api.openai.com/v1/chat/completions", headers=headers, json=payload)
        response.raise_for_status()
        rewrite = response.json()['choices'][0]['message']['content']
        return jsonify({"rewrite": rewrite})
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True, port=5000)
# main
