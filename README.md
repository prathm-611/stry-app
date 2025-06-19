1. app.py — Main Python Flask App  (##basic web version )

   
from flask import Flask, render_template, request, redirect, url_for
from datetime import datetime
import json
import os

app = Flask(__name__)
STORIES_FILE = "stories.json"
current_user = {"username": "guest"}

def load_stories():
    if not os.path.exists(STORIES_FILE):
        return []
    with open(STORIES_FILE, "r") as f:
        return json.load(f)

def save_stories(stories):
    with open(STORIES_FILE, "w") as f:
        json.dump(stories, f, indent=2)

@app.route("/")
def home():
    stories = load_stories()
    return render_template("index.html", stories=stories)

@app.route("/submit", methods=["GET", "POST"])
def submit():
    if request.method == "POST":
        title = request.form["title"]
        content = request.form["content"]

        stories = load_stories()
        new_story = {
            "id": len(stories) + 1,
            "title": title,
            "content": content,
            "author": current_user["username"],
            "timestamp": datetime.now().isoformat(),
            "comments": [],
            "reactions": {"❤️": 0, "😂": 0, "😮": 0, "😢": 0}
        }

        stories.append(new_story)
        save_stories(stories)
        return redirect(url_for("home"))

    return render_template("submit.html")

@app.route("/comment/<int:story_id>", methods=["POST"])
def comment(story_id):
    comment = request.form["comment"].strip()
    stories = load_stories()

    for story in stories:
        if story["id"] == story_id:
            if len(comment.split()) <= 15:
                story["comments"].append(comment)
            else:
                return "⚠️ Comment too long! Max 15 words.", 400
            break

    save_stories(stories)
    return redirect(url_for("home"))

@app.route("/react/<int:story_id>", methods=["POST"])
def react(story_id):
    reaction = request.form["reaction"]
    stories = load_stories()

    for story in stories:
        if story["id"] == story_id:
            if reaction in story["reactions"]:
                story["reactions"][reaction] += 1
            break

    save_stories(stories)
    return redirect(url_for("home"))

if __name__ == "__main__":
    app.run(debug=True)

 templates/base.html


 <!DOCTYPE html>
<html>
<head>
  <title>Storytelling App</title>
</head>
<body>
  <nav>
    <a href="/">🏠 Home</a> |
    <a href="/submit">✍️ Submit a Story</a>
  </nav>
  <hr>
  {% block content %}{% endblock %}
</body>
</html>


templates/index.html

{% extends "base.html" %}
{% block content %}
<h1>📖 Story Wall</h1>

{% for story in stories %}
  <div style="border: 1px solid #ccc; padding: 10px; margin-bottom: 15px;">
    <h3>{{ story.title }}</h3>
    <p>{{ story.content }}</p>
    <p><i>By {{ story.author }} on {{ story.timestamp[:10] }}</i></p>

    <!-- Comments -->
    <h4>💬 Comments</h4>
    {% if story.comments %}
      <ul>
        {% for comment in story.comments %}
          <li>{{ comment }}</li>
        {% endfor %}
      </ul>
    {% else %}
      <p>No comments yet.</p>
    {% endif %}

    <!-- Comment Form -->
    <form method="post" action="/comment/{{ story.id }}">
      <input type="text" name="comment" placeholder="Write a comment (max 15 words)" required>
      <button type="submit">💬 Add Comment</button>
    </form>

    <!-- Reactions -->
    <h4>React:</h4>
    <form method="post" action="/react/{{ story.id }}">
      ❤️ {{ story.reactions["❤️"] }}
      <button type="submit" name="reaction" value="❤️">❤️</button>

      😂 {{ story.reactions["😂"] }}
      <button type="submit" name="reaction" value="😂">😂</button>

      😮 {{ story.reactions["😮"] }}
      <button type="submit" name="reaction" value="😮">😮</button>

      😢 {{ story.reactions["😢"] }}
      <button type="submit" name="reaction" value="😢">😢</button>
    </form>
  </div>
{% endfor %}

{% endblock %}

templates/submit.html

{% extends "base.html" %}
{% block content %}
<h2>Submit a Story</h2>
<form method="post">
  Title: <input type="text" name="title" required><br><br>
  Story:<br>
  <textarea name="content" rows="5" cols="50" required></textarea><br><br>
  <input type="submit" value="Submit">
</form>
{% endblock %}


stories.json (Blank File to Store Data):

touch stories.json

Start the Flask app:

python3 app.py

Open your browser and go to:

http://127.0.0.1:5000


