https://moshpro.app/lite/

'''
import imageio.v2 as imageio
from PIL import Image, ImageSequence
import cv2

def resize_frames(frames, size):
    return [frame.resize(size, Image.ANTIALIAS).convert("RGB") for frame in frames]


# ==== 參數設定 ====
gif1_path = 'gif1.gif'
video_path = 'clip2.mp4'
gif3_path = 'gif3.gif'
output_path = 'combined.gif'
video_cut_seconds = 0.5
video_output_fps = 20

# ==== 工具函式 ====
def extract_gif_frames(path):
    img = Image.open(path)
    frames = []
    durations = []
    for frame in ImageSequence.Iterator(img):
        frames.append(frame.convert('RGBA'))
        durations.append(frame.info.get('duration', 100) / 1000.0)
    return frames, durations

def extract_video_frames(path, cut_duration, fps=20, size=(512, 512)):
    cap = cv2.VideoCapture(path)
    video_fps = cap.get(cv2.CAP_PROP_FPS)
    frames_needed = int(cut_duration * fps)
    frames = []
    durations = []

    frame_interval = int(video_fps / fps) if video_fps > fps else 1
    count = 0
    collected = 0
    while collected < frames_needed:
        ret, frame = cap.read()
        if not ret:
            break
        if count % frame_interval == 0:
            frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            pil_img = Image.fromarray(frame).resize(size)
            frames.append(pil_img)
            durations.append(1.0 / fps)
            collected += 1
        count += 1
    cap.release()
    return frames, durations

# ==== 萃取每段幀與播放時間 ====
gif1_frames, gif1_durations = extract_gif_frames(gif1_path)
video_frames, video_durations = extract_video_frames(video_path, video_cut_seconds)
gif3_frames, gif3_durations = extract_gif_frames(gif3_path)

# ✅ 如果要讓 gif1 播兩次，加這兩行
gif1_frames = gif1_frames * 2
gif1_durations = gif1_durations * 2

# ==== 強制 resize 所有幀為一致大小 ====
target_size = (512, 512)
gif1_frames = resize_frames(gif1_frames, target_size)
video_frames = resize_frames(video_frames, target_size)
gif3_frames = resize_frames(gif3_frames, target_size)

# ==== 合併並輸出 ====
all_frames = gif1_frames + video_frames + gif3_frames
all_durations = gif1_durations + video_durations + gif3_durations

imageio.mimsave(output_path, all_frames, duration=all_durations, loop=0)
print(f"✅ 已輸出：{output_path}")
'''

![Screenshot from 2025-05-31 19-31-00](https://github.com/user-attachments/assets/4f956c61-0b81-4f20-81e5-12cdd1eeada3)


