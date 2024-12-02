# SongScripts
# SongScripts
# app.py (Flask Backend)
from flask import Flask, request, jsonify, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///music.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)

class Song(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(120), nullable=False)
    artist = db.Column(db.String(120), nullable=False)

class Playlist(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    songs = db.relationship('Song', secondary='playlist_song', backref='playlists')

playlist_song = db.Table('playlist_song',
    db.Column('playlist_id', db.Integer, db.ForeignKey('playlist.id'), primary_key=True),
    db.Column('song_id', db.Integer, db.ForeignKey('song.id'), primary_key=True)
)

@app.before_first_request
def create_tables():
    db.create_all()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/add_song', methods=['POST'])
def add_song():
    data = request.json
    new_song = Song(title=data['title'], artist=data['artist'])
    db.session.add(new_song)
    db.session.commit()
    return jsonify({'message': 'Song added successfully'}), 201

@app.route('/create_playlist', methods=['POST'])
def create_playlist():
    data = request.json
    user = User.query.first()  # Assuming a single user for simplicity
    if not user:
        user = User(username="default_user")
        db.session.add(user)
        db.session.commit()

    new_playlist = Playlist(name=data['name'], user_id=user.id)
    db.session.add(new_playlist)
    db.session.commit()
    return jsonify({'message': 'Playlist created successfully'}), 201

@app.route('/add_song_to_playlist', methods=['POST'])
def add_song_to_playlist():
    data = request.json
    playlist = Playlist.query.get(data['playlist_id'])
    song = Song.query.get(data['song_id'])
    if not playlist or not song:
        return jsonify({'message': 'Playlist or Song not found'}), 404

    playlist.songs.append(song)
    db.session.commit()
    return jsonify({'message': 'Song added to playlist successfully'}), 200

if __name__ == '__main__':
    app.run(debug=True)

<!-- templates/index.html (Frontend) -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Music App</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
</head>
<body>
    <h1>Music App</h1>
    
    <!-- Add Song Form -->
    <form id="songForm">
        <input type="text" id="songTitle" placeholder="Song Title" required>
        <input type="text" id="songArtist" placeholder="Artist" required>
        <button type="submit">Add Song</button>
    </form>

    <!-- Create Playlist Form -->
    <form id="playlistForm">
        <input type="text" id="playlistName" placeholder="Playlist Name" required>
        <button type="submit">Create Playlist</button>
    </form>

    <script>
        document.getElementById('songForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const title = document.getElementById('songTitle').value;
            const artist = document.getElementById('songArtist').value;

            axios.post('/add_song', { title, artist })
                .then(response => alert(response.data.message))
                .catch(error => console.error(error));
        });

        document.getElementById('playlistForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const name = document.getElementById('playlistName').value;

            axios.post('/create_playlist', { name })
                .then(response => alert(response.data.message))
                .catch(error => console.error(error));
        });
    </script>
</body>
</html>
