import time
import pygetwindow as gw
import pyautogui
from PIL import Image
import pytesseract
import openai
import io
import mouse  # للتعامل مع ضغطات الماوس
import tkinter as tk
from tkinter import simpledialog

# إعداد Tesseract
pytesseract.pytesseract.tesseract_cmd = r'مسار_Tesseract_على_جهازك'

# إعداد متصفح Selenium
driver_path = 'مسار_WebDriver_على_جهازك'
driver = None  # متغير المتصفح سيكون فارغًا إلى أن يتم فتح المتصفح

# إعداد OpenAI
openai.api_key = "مفتاح_التطبيق_من_OpenAI"

# وظيفة لالتقاط صورة للصفحة وتحليل النص
def capture_and_analyze():
    # التقاط صورة للشاشة للنافذة الحالية
    screenshot = pyautogui.screenshot()
    image = Image.open(io.BytesIO(screenshot.tobytes()))

    # استخدم OCR لاستخراج النص من الصورة
    extracted_text = pytesseract.image_to_string(image)
    return extracted_text

# إرسال النص المستخرج إلى ChatGPT
def send_to_chatgpt(text):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "user", "content": text}
        ]
    )
    return response.choices[0].message['content']

# عرض الإجابة في زاوية الشاشة
def show_answer_in_corner(answer):
    root = tk.Tk()
    root.title("إجابة ChatGPT")
    root.geometry("300x150+1000+100")  # تحديد الحجم والموقع في الزاوية
    label = tk.Label(root, text=answer, wraplength=250)
    label.pack(padx=10, pady=10)
    root.after(5000, lambda: root.destroy())  # إغلاق الإجابة بعد 5 ثواني
    root.mainloop()

# وظيفة لتشغيل الأداة عند النقر بزر الماوس الأيمن
def monitor_mouse():
    print("انتظر، اضغط بزر الماوس الأيمن لتشغيل الأداة.")
    
    while True:
        if mouse.is_pressed(button='right'):
            print("تم الضغط بزر الماوس الأيمن - بدء تشغيل الأداة.")
            # عرض مربع حوار لتحديد النص الذي سيتم نسخه
            root = tk.Tk()
            root.withdraw()  # إخفاء النافذة الرئيسية

            # نافذة لتحديد النص المطلوب
            user_input = simpledialog.askstring("أدخل النص", "أدخل النص الذي تريد نسخه إلى ChatGPT:")
            
            if user_input:
                print("النص المدخل:", user_input)

                # إرسال النص إلى ChatGPT
                response = send_to_chatgpt(user_input)
                print("الإجابة من ChatGPT:", response)

                # عرض الإجابة في زاوية الشاشة
                show_answer_in_corner(response)
            root.destroy()

        # الانتظار قليلا قبل التحقق من ضغط الماوس مرة أخرى
        time.sleep(0.1)

# بدء مراقبة زر الماوس
monitor_mouse()
