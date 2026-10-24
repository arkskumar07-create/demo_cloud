# demo_cloud




import html
import json
import os
import urllib.parse
import urllib.request
from datetime import datetime

import streamlit as st

# ---------- 1. Page settings (this MUST be the first Streamlit command) ----------
st.set_page_config(
    page_title="Keshav Kumar Singh | Class 8 Study",
    page_icon="📚",
    layout="wide",
)

COMMENTS_FILE = "comments.json"
COMMENTS_PAGE = "💬 Comments"

# ---------- 2. Your video folders ----------
# Just paste the YouTube links. The titles are fetched from YouTube automatically.
FOLDERS = {
    "🔬 Science": [
        "https://youtu.be/FU7r5EebVh4?si=d2z0rOL8-C1fJAQd",
        "https://youtu.be/qiCqiLbmH0Y?si=bZxFQ4ZZR-oOz1jn",
    ],
    "➗ Maths": [
        "https://youtu.be/4lADI2NCL6I?si=39aR9Bf2mAYj1o-H",
        "https://youtu.be/qyxwyomDjT8?si=KNcDxjzbCLXrvN0m",
    ],
    "🌍 SST": [
        "https://youtu.be/EcGgE_ryWGU?si=whxfBan6Iy8QaC19",
        "https://youtu.be/y1tadOF9Fa0?si=rT_jn4-MAWoIklSk",
        "https://youtu.be/SBVmEVOolH8?si=Vqt-fXtRogLH5dtq",
        "https://youtu.be/FkN3GJRennI?si=DKHbHmS5zTFz_Kxc",
        "https://youtu.be/S06BiFb19yo?si=rml7kaOHSBUkKR-u",
        "https://youtu.be/K5idy7UlMBA?si=JSk4TKtXj8YCUGXB",
    ],
}

# ---------- 3. Colour themes ----------
# Each theme is a set of colours. Add your own theme by copying one line!
THEMES = {
    "☀️ Light": dict(bg="#ffffff", bg2="#f3f4f8", text="#1f2430", muted="#5b6475",
                    accent="#6a11cb", accent2="#2575fc", input_bg="#ffffff", border="#d5d9e3"),
    "🌙 Dark": dict(bg="#0e1117", bg2="#1a1f2b", text="#f1f3f8", muted="#aab2c3",
                   accent="#8b5cf6", accent2="#3b82f6", input_bg="#262c3a", border="#343b4d"),
    "🌊 Ocean": dict(bg="#0b2239", bg2="#12395c", text="#e8f4ff", muted="#a9c8e4",
                    accent="#0096c7", accent2="#0077b6", input_bg="#0f3050", border="#1e5a8a"),
    "🌲 Forest": dict(bg="#0f2418", bg2="#17382a", text="#e8f5ec", muted="#a8cdb4",
                     accent="#27ae60", accent2="#1e8449", input_bg="#143023", border="#25593f"),
    "🌅 Sunset": dict(bg="#fff4e8", bg2="#ffe3cc", text="#3a1f12", muted="#7a4a33",
                     accent="#ff6b35", accent2="#e63946", input_bg="#ffffff", border="#f2b996"),
    "💜 Purple Night": dict(bg="#1a1030", bg2="#2a1b4d", text="#f3ebff", muted="#c3b0e6",
                           accent="#a855f7", accent2="#7b2cbf", input_bg="#231545", border="#4a2f82"),
    "🌸 Pink": dict(bg="#fff0f6", bg2="#ffdbe9", text="#3d1429", muted="#85405f",
                   accent="#ff4d94", accent2="#c2185b", input_bg="#ffffff", border="#f5b3cf"),
}

# The theme chosen by the user (saved in session_state under the key "theme")
theme_name = st.session_state.get("theme", "☀️ Light")
t = THEMES[theme_name]

# Step 1: turn the theme colours into CSS variables
theme_css = f"""
<style>
:root {{
    --bg: {t['bg']};
    --bg2: {t['bg2']};
    --text: {t['text']};
    --muted: {t['muted']};
    --accent: {t['accent']};
    --accent2: {t['accent2']};
    --input-bg: {t['input_bg']};
    --border: {t['border']};
}}
</style>
"""

# Step 2: the style rules that use those variables
main_css = """
<style>
/* ----- Page and sidebar colours ----- */
.stApp, [data-testid="stAppViewContainer"] { background: var(--bg) !important; }
[data-testid="stHeader"] { background: var(--bg) !important; }
[data-testid="stHeader"], [data-testid="stHeader"] * { color: var(--text) !important; }
[data-testid="stSidebar"], [data-testid="stSidebar"] > div { background: var(--bg2) !important; }

/* ----- Text colours ----- */
.stApp, .stApp p, .stApp li, .stApp label, .stApp span,
.stApp h1, .stApp h2, .stApp h3, .stApp h4, .stApp h5, .stApp h6 { color: var(--text) !important; }
.stApp [data-testid="stCaptionContainer"],
.stApp [data-testid="stCaptionContainer"] p { color: var(--muted) !important; }
.stApp hr { border-color: var(--border) !important; }

/* ----- Banner at the top ----- */
.hero {
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    padding: 45px 30px;
    border-radius: 20px;
    text-align: center;
    margin-bottom: 25px;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.25);
}
.stApp .hero h1 { color: #ffffff !important; font-size: 2.4rem; margin-bottom: 8px; }
.stApp .hero p  { color: #ffffff !important; font-size: 1.2rem; margin: 0; opacity: 0.95; }

/* ----- Buttons (folder buttons and Post comment) ----- */
.stApp .stButton > button,
.stApp [data-testid="stFormSubmitButton"] > button {
    width: 100%;
    border-radius: 14px;
    padding: 14px 10px;
    border: 2px solid var(--accent);
    background: var(--bg2);
    color: var(--text);
    transition: transform 0.2s, box-shadow 0.2s;
}
.stApp .stButton > button p,
.stApp [data-testid="stFormSubmitButton"] > button p {
    color: inherit !important;
    font-weight: 700;
    font-size: 1.05rem;
    margin: 0;
}
.stApp .stButton > button:hover,
.stApp [data-testid="stFormSubmitButton"] > button:hover {
    transform: translateY(-4px);
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.25);
    border-color: var(--accent2);
}
/* The selected folder button and the Post button are filled with colour */
.stApp .stButton > button[kind="primary"],
.stApp .stButton > button[data-testid="stBaseButton-primary"],
.stApp [data-testid="stFormSubmitButton"] > button {
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    color: #ffffff;
    border-color: transparent;
}

/* ----- Text boxes and the form ----- */
.stApp [data-baseweb="input"],
.stApp [data-baseweb="textarea"],
.stApp [data-baseweb="base-input"] {
    background-color: var(--input-bg) !important;
    border-color: var(--border) !important;
}
.stApp input, .stApp textarea {
    background-color: var(--input-bg) !important;
    color: var(--text) !important;
    -webkit-text-fill-color: var(--text) !important;
}
[data-testid="stForm"] {
    background: var(--bg2) !important;
    border: 1px solid var(--border) !important;
    border-radius: 16px;
}
[data-testid="stVerticalBlockBorderWrapper"] { border-color: var(--border) !important; }

/* ----- Theme dropdown (selectbox) ----- */
.stApp [data-baseweb="select"] > div {
    background-color: var(--input-bg) !important;
    border-color: var(--border) !important;
}
.stApp [data-baseweb="select"] div,
.stApp [data-baseweb="select"] span,
.stApp [data-baseweb="select"] input { color: var(--text) !important; }
[data-baseweb="popover"] ul,
[data-baseweb="popover"] [role="listbox"],
[data-baseweb="menu"] { background-color: var(--bg2) !important; }
[data-baseweb="popover"] li,
[data-baseweb="popover"] [role="option"] {
    background-color: var(--bg2) !important;
    color: var(--text) !important;
}
[data-baseweb="popover"] li:hover,
[data-baseweb="popover"] [role="option"]:hover,
[data-baseweb="popover"] [aria-selected="true"] {
    background-color: var(--accent) !important;
    color: #ffffff !important;
}
[data-baseweb="popover"] li * { color: inherit !important; }

/* ----- Videos ----- */
.stApp iframe { border-radius: 12px; }
.video-title { font-weight: 700; font-size: 1.05rem; margin-bottom: 8px; }

/* ----- Comment cards ----- */
.comment-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-left: 5px solid var(--accent);
    padding: 14px 18px;
    border-radius: 12px;
    margin-bottom: 12px;
}
.comment-name { font-weight: 700; font-size: 1.05rem; }
.comment-time { font-size: 0.8rem; opacity: 0.7; margin-left: 8px; }
.comment-text { margin-top: 6px; }

/* Hide the default Streamlit footer */
footer { visibility: hidden; }
</style>
"""

st.markdown(theme_css + main_css, unsafe_allow_html=True)


# ---------- 4. Helper functions ----------
@st.cache_data(ttl=3600, show_spinner=False)
def get_youtube_title(url):
    """Ask YouTube for the video title (saved for 1 hour so it is fast)."""
    api = "https://www.youtube.com/oembed?format=json&url=" + urllib.parse.quote(url, safe="")
    with urllib.request.urlopen(api, timeout=6) as response:
        return json.loads(response.read().decode("utf-8"))["title"]


def video_title(url, number):
    """Return the YouTube title, or 'Lesson N' if YouTube can't be reached."""
    try:
        return get_youtube_title(url)
    except Exception:
        return f"Lesson {number}"


def show_videos(urls):
    """Show videos in 2 columns, each with its YouTube title."""
    for i in range(0, len(urls), 2):
        col_a, col_b = st.columns(2)
        for offset, col in enumerate((col_a, col_b)):
            if i + offset >= len(urls):
                break
            url = urls[i + offset]
            title = video_title(url, i + offset + 1)
            with col:
                with st.container(border=True):
                    st.markdown(
                        f"<div class='video-title'>▶️ {html.escape(title)}</div>",
                        unsafe_allow_html=True,
                    )
                    st.video(url)


def load_comments():
    """Read all saved comments from the file (or return an empty list)."""
    if os.path.exists(COMMENTS_FILE):
        with open(COMMENTS_FILE, "r") as f:
            return json.load(f)
    return []


def save_comments(comments):
    """Write the list of comments to the file."""
    with open(COMMENTS_FILE, "w") as f:
        json.dump(comments, f, indent=2)


def go_to(page):
    """Called when a folder button is clicked."""
    st.session_state.page = page


# ---------- 5. Sidebar ----------
with st.sidebar:
    st.title("📚 Menu")
    st.write("Welcome, students of **Class 8**!")
    st.divider()
    st.markdown("**Video folders**")
    for folder_name, urls in FOLDERS.items():
        st.write(f"{folder_name} ({len(urls)} videos)")
    st.divider()
    st.info("Click a folder button, watch the videos, then leave a comment. Keep studying! 💪")

# ---------- 6. Hero banner ----------
st.markdown(
    """
    <div class="hero">
        <h1>🎓 Keshav Kumar Singh</h1>
        <p>YouTube Channel &nbsp;|&nbsp; Study for Class 8th</p>
    </div>
    """,
    unsafe_allow_html=True,
)

# ---------- 7. Theme chooser + folder buttons ----------
top_left, top_right = st.columns([3, 1])
with top_left:
    st.subheader("📂 Choose a folder")
with top_right:
    st.selectbox("🎨 Theme", list(THEMES.keys()), key="theme")

NAV_ITEMS = list(FOLDERS.keys()) + [COMMENTS_PAGE]

if "page" not in st.session_state:
    st.session_state.page = NAV_ITEMS[0]  # start on the first folder

button_columns = st.columns(len(NAV_ITEMS))
for col, item in zip(button_columns, NAV_ITEMS):
    label = f"{item} ({len(FOLDERS[item])})" if item in FOLDERS else item
    with col:
        st.button(
            label,
            key=f"nav_{item}",
            type="primary" if st.session_state.page == item else "secondary",
            on_click=go_to,
            args=(item,),
        )

st.write("")  # empty space

# ---------- 8. Show the page the user selected ----------
page = st.session_state.page

if page in FOLDERS:
    # ----- A video folder -----
    st.subheader(f"{page} Folder")
    st.caption(f"{len(FOLDERS[page])} videos in this folder")
    show_videos(FOLDERS[page])

else:
    # ----- The comments page -----
    st.subheader("💬 Leave a comment")
    st.write("Tell me what you think about the videos!")

    with st.form("comment_form", clear_on_submit=True):
        name = st.text_input("Your name")
        text = st.text_area("Your comment")
        submitted = st.form_submit_button("🚀 Post comment")

    if submitted:
        if name.strip() == "" or text.strip() == "":
            st.warning("Please fill in both your name and your comment.")
        else:
            comments = load_comments()
            comments.append(
                {
                    "name": name.strip(),
                    "text": text.strip(),
                    "time": datetime.now().strftime("%d %b %Y, %H:%M"),
                }
            )
            save_comments(comments)
            st.success("Thanks for your comment!")
            st.balloons()

    st.divider()
    st.subheader("🗨️ All comments")

    comments = load_comments()

    if len(comments) == 0:
        st.info("No comments yet. Be the first!")
    else:
        st.caption(f"{len(comments)} comment(s)")
        for c in reversed(comments):  # newest first
            # html.escape keeps the page safe if someone types HTML code
            safe_name = html.escape(c["name"])
            safe_text = html.escape(c["text"]).replace("\n", "<br>")
            st.markdown(
                f"""
                <div class="comment-card">
                    <span class="comment-name">👤 {safe_name}</span>
                    <span class="comment-time">{c['time']}</span>
                    <div class="comment-text">{safe_text}</div>
                </div>
                """,
                unsafe_allow_html=True,
            )

# ---------- 9. Footer ----------
st.divider()
st.markdown(
    "<p style='text-align:center; opacity:0.7;'>Made with ❤️ by Keshav Kumar Singh · Keep learning, keep growing!</p>",
    unsafe_allow_html=True,
)
