#work in progress
#Before running the code, you will need to install the library via your terminal:

#bash
pip install scenedetect[opencv] pandas

#You'll need to install the ReportLab library:
pip install reportlab

#start code(python)
#This script will analyze a video file, detect the cuts, and save a clean .csv report to your directory.

import csv
from scenedetect import open_video, SceneManager, split_video_ffmpeg
from scenedetect.detectors import ContentDetector
from scenedetect.video_manager import VideoManager

def generate_timecode_doc(video_path, output_csv):
    print(f"Processing video: {video_path}...")
    
    # 1. Open the video file
    video = open_video(video_path)
    
    # 2. Initialize SceneManager
    scene_manager = SceneManager()
    
    # 3. Add ContentDetector
    # 'threshold' determines sensitivity (27.0 is standard). 
    # Lower = more sensitive (detects subtle fades). Higher = strict cuts only.
    scene_manager.add_detector(ContentDetector(threshold=27.0))
    
    # 4. Perform scene detection
    scene_manager.detect_scenes(video, show_progress=True)
    
    # 5. Get list of scenes (tuples of FrameTimecode objects)
    scene_list = scene_manager.get_scene_list()
    
    print(f"Detection complete. Found {len(scene_list)} scenes. Writing to CSV...")

    # 6. Write to CSV
    with open(output_csv, mode='w', newline='') as file:
        writer = csv.writer(file)
        
        # Header Row
        writer.writerow(['Scene Number', 'Start Timecode', 'End Timecode', 'Duration'])
        
        for i, scene in enumerate(scene_list):
            start, end = scene
            scene_num = i + 1
            
            # Format: HH:MM:SS.nnn
            start_tc = start.get_timecode() 
            end_tc = end.get_timecode()
            duration = end - start
            
            writer.writerow([scene_num, start_tc, end_tc, duration.get_timecode()])

    print(f"Documentation saved to: {output_csv}")

# --- CONFIGURATION ---
if __name__ == "__main__":
    # Replace with your actual video file path
    INPUT_VIDEO = "my_video_file.mp4" 
    OUTPUT_REPORT = "scene_timecodes.csv"

   
    try:
        generate_timecode_doc(INPUT_VIDEO, OUTPUT_REPORT)
    except Exception as e:
        print(f"Error: {e}")
        print("Ensure the video path is correct and ffmpeg/opencv dependencies are installed.")



#Python Code for PDF Generation
This script reads your previously generated visual_scene_report.csv and the JPEG files from the scene_thumbnails folder to build the final document.

import csv
import os
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Image

# --- CONFIGURATION (Must match previous script's output) ---
PDF_OUTPUT = "Scene_Analysis_Report.pdf"
CSV_INPUT = "visual_scene_report.csv"
IMAGES_DIR = "scene_thumbnails"
IMAGE_WIDTH = 250  # Width in ReportLab units (e.g., points)
IMAGE_HEIGHT = 140  # Maintain aspect ratio for standard video formats

def generate_pdf_report():
    print(f"Starting PDF generation: {PDF_OUTPUT}")
    
    # 1. Initialize PDF Document
    doc = SimpleDocTemplate(PDF_OUTPUT, pagesize=A4, 
                            rightMargin=40, leftMargin=40, 
                            topMargin=40, bottomMargin=40)
    
    styles = getSampleStyleSheet()
    story = [] # List to hold all PDF elements (Paragraphs, Images, etc.)

    # 2. Add Title and Header
    story.append(Paragraph("🎬 Automated Scene Timecode Report", styles['Title']))
    story.append(Spacer(1, 12))
    story.append(Paragraph(f"Source Data: {CSV_INPUT}", styles['Normal']))
    story.append(Spacer(1, 24))

    # 3. Read CSV and Build Content
    if not os.path.exists(CSV_INPUT):
        print(f"Error: CSV file not found at {CSV_INPUT}")
        return

    with open(CSV_INPUT, mode='r', newline='') as file:
        reader = csv.reader(file)
        next(reader) # Skip header row
        
        for row in reader:
            scene_num, start_tc, end_tc, duration, image_filename = row
            
            # --- Text Content (Timecodes) ---
            text_content = f"""
            <b>Scene #</b>: {scene_num}<br/>
            <b>START TC</b>: {start_tc}<br/>
            <b>END TC</b>: {end_tc}<br/>
            <b>Duration</b>: {duration}
            """
            story.append(Paragraph(text_content, styles['Normal']))
            story.append(Spacer(1, 6))

            # --- Image Preview ---
            image_path = os.path.join(IMAGES_DIR, image_filename)
            if os.path.exists(image_path):
                img = Image(image_path, IMAGE_WIDTH, IMAGE_HEIGHT)
                story.append(img)
            else:
                story.append(Paragraph(f"<b>[ERROR: Image Missing: {image_filename}]</b>", styles['Italic']))
            
            # Add separator line between scenes
            story.append(Spacer(1, 18))

    # 4. Generate the PDF
    doc.build(story)
    print(f"Successfully generated PDF report: {PDF_OUTPUT}")

if __name__ == "__main__":
    generate_pdf_report()

#The resulting PDF will be clean and structured, showing the critical metadata directly above the visual reference.
#This layout allows an editor or director to quickly scan the document for scene cuts and visually verify the exact moment of transition.
