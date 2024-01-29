# Lucasi
Lucasi style program merge


Create a better interface by mixing these two programs:  import streamlit as st
import cv2
import numpy as np
import datetime
import os
import time
import base64
from camera_input_live import camera_input_live

def save_image(image):
    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"captured_image_{timestamp}.png"
    bytes_data = image.getvalue()
    cv2_img = cv2.imdecode(np.frombuffer(bytes_data, np.uint8), cv2.IMREAD_COLOR)
    cv2.imwrite(filename, cv2_img)
    return filename

def get_image_base64(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode()

def main():
    st.title("Streamlit Camera Real Time Live Streming")
    st.header("QR code reader ready.")

    # Slider for snapshot interval
    snapshot_interval = st.sidebar.slider("Snapshot Interval (seconds)", 1, 10, 5)

    # Display container for live image
    image_placeholder = st.empty()

    # Initialize session state for captured images and last captured time
    if 'captured_images' not in st.session_state:
        st.session_state['captured_images'] = []
    if 'last_captured' not in st.session_state:
        st.session_state['last_captured'] = time.time()

    # Capture and display live image
    image = camera_input_live()
    if image is not None:
        image_placeholder.image(image)

        # Save image based on interval
        if time.time() - st.session_state['last_captured'] > snapshot_interval:
            filename = save_image(image)
            st.session_state['captured_images'].append(filename)
            st.session_state['last_captured'] = time.time()

    # Update sidebar with captured images
    sidebar_html = "<div style='display:flex;flex-direction:column;'>"
    for img_file in st.session_state['captured_images']:
        image_base64 = get_image_base64(img_file)
        sidebar_html += f"<img src='data:image/png;base64,{image_base64}' style='width:100px;'><br>"
    sidebar_html += "</div>"
    st.sidebar.markdown("## Captured Images")
    st.sidebar.markdown(sidebar_html, unsafe_allow_html=True)

    # Trigger a rerun of the app to update the live feed
    st.experimental_rerun()

if __name__ == "__main__":
    main()
  and program 2 is:  import streamlit as st
import streamlit.components.v1 as components
import re
import os
import glob
st.set_page_config(layout="wide")
def process_line(line):
    if re.search(r'\b[A-G][#b]?m?\b', line):
        line = re.sub(r'\b([A-G][#b]?m?)\b', r"<img src='\1.png' style='height:20px;'>", line)
    return line
def process_chord_sheet(chord_sheet):
    processed_lines = []
    for line in chord_sheet.split('\n'):
        processed_line = process_line(line)
        processed_lines.append(processed_line)
    return '<br>'.join(processed_lines)
def create_search_url_wikipedia(artist_song):
    base_url = "https://www.wikipedia.org/search-redirect.php?family=wikipedia&language=en&search="
    return base_url + artist_song.replace(' ', '+').replace('–', '%E2%80%93').replace('&', 'and')
def create_search_url_youtube(artist_song):
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + artist_song.replace(' ', '+').replace('–', '%E2%80%93').replace('&', 'and')
def create_search_url_chords(artist_song):
    base_url = "https://www.ultimate-guitar.com/search.php?search_type=title&value="
    return base_url + artist_song.replace(' ', '+').replace('–', '%E2%80%93').replace('&', 'and')
def create_search_url_lyrics(artist_song):
    base_url = "https://www.google.com/search?q="
    return base_url + artist_song.replace(' ', '+').replace('–', '%E2%80%93').replace('&', 'and') + '+lyrics'
def songupdate():
    st.write(st.session_state.EnhancedChordSheet)
def load_song_file2(filename):
    with open(filename, "r") as file:
        chord_sheet = file.read()
        st.session_state['chord_sheet'] = chord_sheet
        processed_sheet = process_chord_sheet(chord_sheet)
        st.markdown(processed_sheet, unsafe_allow_html=True)
def load_song_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        chord_sheet = file.read()
    return chord_sheet
def song_update():
    if 'selected_file' in st.session_state:
        song_name, artist_name = parse_filename(st.session_state.selected_file)
        st.session_state.song_name = song_name
        st.session_state.artist_name = artist_name
def parse_filename(filename):
    base_name = os.path.splitext(filename)[0]
    song_name, artist_name = base_name.split(' by ')
    return song_name.replace("_", " "), artist_name.replace("_", " ")
def auto_save():
    song_name = st.session_state.get('song_name', '')
    artist_name = st.session_state.get('artist_name', '')
    chord_sheet = st.session_state.get('chord_sheet', '')
    if song_name and artist_name and chord_sheet:
        filename = song_name + " by " + artist_name + ".txt"
        with open(filename, "w") as file:
            chord_sheet_text = st.session_state.get('chord_sheet', '')
            file.write(chord_sheet_text)
        st.session_state['char_count'] = len(chord_sheet)
        st.success(f"Auto-saved to {filename}")
def main():
    col1, col3 = st.columns([3, 5])
    with col1:
        st.markdown('### 🎵 📚Prompt🎥🎸Chord Sheet🎶 AI Prompt Authoring App')
        with st.expander("Select Song:", expanded=True):
            all_files = [f for f in glob.glob("*.txt") if ' by ' in f]
            selected_file = st.selectbox("Choose: ", all_files, on_change=song_update, key='selected_file')
        song_name_input = st.text_input("🎵 Song:", key='song_name', on_change=auto_save)
        artist_name_input = st.text_input("🎤 Artist:", key='artist_name', on_change=auto_save)
        if 'selected_file' in st.session_state and st.session_state.selected_file:
            # Update the session state before creating the text area widget
            st.session_state['chord_sheet'] = load_song_file(st.session_state.selected_file)
        st.header("🎼 Current Song")
        load_song_file(selected_file)
        song_info = os.path.splitext(selected_file)[0].replace("_", " ")
        st.markdown("**" + song_info + "**")
        table_md = f"""
        | Wikipedia | YouTube | Chords | Lyrics |
        | --------- | ------- | ------ | ------ |
        | [📚]({create_search_url_wikipedia(song_info)}) | [🎥]({create_search_url_youtube(song_info)}) | [🎸]({create_search_url_chords(song_info)}) | [🎶]({create_search_url_lyrics(song_info)}) |
        """
        st.markdown(table_md)
        st.header("🎼 Available Songs")
        for file in all_files:
            song_info = os.path.splitext(file)[0].replace("_", " ")
            icol1, icol2 = st.columns([1, 3])
            with icol1:
                st.markdown("**" + song_info + "**")
                load_song_file(file)
                song_info = os.path.splitext(file)[0].replace("_", " ")
            with icol2:
                # Create a markdown table with links for each song file
                table_md = f"""
                | Wikipedia | YouTube | Chords | Lyrics |
                | --------- | ------- | ------ | ------ |
                | [📚]({create_search_url_wikipedia(song_info)}) | [🎥]({create_search_url_youtube(song_info)}) | [🎸]({create_search_url_chords(song_info)}) | [🎶]({create_search_url_lyrics(song_info)}) |
                """
                st.markdown(table_md)
    with col3:
        subcol1, subcol2 = st.columns([1, 5])
        with subcol2:
            chord_sheet_area = st.text_area("Chord Sheet", value=st.session_state.get('chord_sheet', ''), height=1600, key='chord_sheet', on_change=auto_save)
        with subcol1:
            # Save functionality
            if st.button("💾 Save", key="save_song"):
                if song_name_input and artist_name_input:
                    filename = song_name_input + " by " + artist_name_input + ".txt"
                    with open(filename, "w") as file:
                        file.write(chord_sheet_area)
                    st.success("Chord sheet saved to file: " + filename)
                else:
                    st.error("Both Song Name and Artist Name are required.")
            char_count_msg = f"Character Count: {st.session_state.get('char_count', 0)}"
            st.write(char_count_msg)
    # Load chord sheet from selected file into the text area
    if 'selected_file' in st.session_state and st.session_state.selected_file:
        load_song_file(st.session_state.selected_file)
if __name__ == '__main__':
    main()


import streamlit as st
import cv2
import numpy as np
import datetime
import os
import time
import base64
import re
import glob
from camera_input_live import camera_input_live

# Set wide layout
st.set_page_config(layout="wide")

# Function Definitions for Camera Feature
def save_image(image):
    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"captured_image_{timestamp}.png"
    bytes_data = image.getvalue()
    cv2_img = cv2.imdecode(np.frombuffer(bytes_data, np.uint8), cv2.IMREAD_COLOR)
    cv2.imwrite(filename, cv2_img)
    return filename

def get_image_base64(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode()

# Function Definitions for Chord Sheet Feature
def process_line(line):
    if re.search(r'\b[A-G][#b]?m?\b', line):
        line = re.sub(r'\b([A-G][#b]?m?)\b', r"<img src='\1.png' style='height:20px;'>", line)
    return line

def process_chord_sheet(chord_sheet):
    processed_lines = []
    for line in chord_sheet.split('\n'):
        processed_line = process_line(line)
        processed_lines.append(processed_line)
    return '<br>'.join(processed_lines)

# Main Function
def main():
    # Layout Configuration
    col1, col2, col3 = st.columns([2, 3, 5])

    # Camera Section
    with col1:
        st.title("Real-Time Camera Stream")
        st.header("QR code reader ready.")

        snapshot_interval = st.slider("Snapshot Interval (seconds)", 1, 10, 5)
        image_placeholder = st.empty()

        if 'captured_images' not in st.session_state:
            st.session_state['captured_images'] = []
        if 'last_captured' not in st.session_state:
            st.session_state['last_captured'] = time.time()

        image = camera_input_live()
        if image is not None:
            image_placeholder.image(image)

            if time.time() - st.session_state['last_captured'] > snapshot_interval:
                filename = save_image(image)
                st.session_state['captured_images'].append(filename)
                st.session_state['last_captured'] = time.time()

        sidebar_html = "<div style='display:flex;flex-direction:column;'>"
        for img_file in st.session_state['captured_images']:
            image_base64 = get_image_base64(img_file)
            sidebar_html += f"<img src='data:image/png;base64,{image_base64}' style='width:100px;'><br>"
        sidebar_html += "</div>"
        st.sidebar.markdown("## Captured Images")
        st.sidebar.markdown(sidebar_html, unsafe_allow_html=True)

    # Chord Sheet Section
    with col2:
        st.title("Chord Sheet Manager")

        all_files = [f for f in glob.glob("*.txt") if ' by ' in f]
        selected_file = st.selectbox("Choose a Song:", all_files)

        if selected_file:
            with open(selected_file, 'r', encoding='utf-8') as file:
                chord_sheet = file.read()
            st.markdown(process_chord_sheet(chord_sheet), unsafe_allow_html=True)

    # Trigger a rerun to update the live feed
    st.experimental_rerun()

if __name__ == "__main__":
    main()
