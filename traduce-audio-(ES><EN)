import os
import sys
import logging
import argparse
from typing import Tuple

import whisper
from moviepy.editor import VideoFileClip
from googletrans import Translator
from gtts import gTTS

# Confirmacion del loggig para mostrar informacion y errores
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Cargar el modelo de whisper una unica vez
model = whisper.load_model("small")

def extract_audio(video, audio_path: str) -> str:
    """
    Extrae el audio del video y lo escribe en formato WAV con la frecuencia adecuada.
    
    Args:
        video: El objeto de video del cual se extraerá el audio.
        audio_path (str): Ruta donde se guardará el archivo de audio.

    Returns:
        str: La ruta del archivo de audio.
    """
    try:
        # Escribir el audio extraído en formato WAV con la frecuencia adecuada.
        video.audio.write_audiofile(audio_path, fps=16000, codec='pcm_s16le')
        return audio_path
    except Exception as e:
        logging.error(f"Error al extraer el audio del video: {e}")
        raise

def transcribe_audio(audio_path: str) -> str:
    """
    Transcribe el audio de un archivo de audio a texto.

    Args:
        audio_path (str): Ruta del archivo de audio.

    Returns:
        str: El texto transcrito.
    """
    # ... tu código para la transcripción ...
    return "texto transcrito"  # Ejemplo de retorno

def translate_text(text: str, target_language: str) -> str:
    """
    Traduce el texto al idioma especificado
    Args:
        text (str): Texto a traducir.
        target_language (str): Idioma al que se traducira el texto.
        
        Returns:
            str: Texto traducido al idioma especificado.
            
            Raises:
                Exception: Si ocurre un error al traducir el texto.
    """
    try:
        logging.info("Traduciendo texto...")
        translator = Translator()
        translation = translator.translate(text, dest=target_language)
    except Exception as e:
        logging.error(f"Error al traducir el texto: {e}")
        raise

    def generate_audio(text: str, lang: str = "es", output_file: str = "translated_audio.mp3") -> str:
        """
        Genera un archivo de audio a partir de un texto.
        Args:
            text (str): Texto a convertir en audio.
            lang (str): Idioma del texto.
            output_file (str): Ruta donde se guardara el archivo de audio.
            
            Returns:
                str: Ruta del archivo de audio generado.
                
                Raises:
                    Exception: Si ocurre un error al generar el archivo de audio.
        """
        try:
            logging.info("Generando archivo de audio...")
            tts = gTTS(text, lang=lang)
            tts.save(output_file)
            return output_file
        except Exception as e:
            logging.error(f"Error al generar el archivo de audio: {e}")
            raise
        def create_str_file(translated_text: str, video_duration: float, output_file: str = "subtitles.srt") -> str:
            """
            Crea un archivo de subtitulos en formato SRT.
            Args:
                translated_text (str): Texto traducido.
                video_duration (float): Duracion del video.
                output_file (str): Ruta donde se guardara el archivo de subtitulos.
                
                Returns:
                    str: Ruta del archivo de subtitulos generado.
                    
                    Raises:
                        Exception: Si ocurre un error al generar el archivo de subtitulos.
            """
            try:
                logging.info("Creando archivo de SRT...")
                # Formatear la duracion del video en el formato SRT: HH:MM:SS,mmm
                hours, remainder = divmod(video_duration, 3600)
                minutes, seconds = divmod(remainder, 60)
                # Para mayor precision se podria formatear los milisegundos.
                end_time = f"{int(hours):02d}:{int(minutes):02d}:{int(seconds):02d},000"
                start_time = "00:00:00,000"

                # Escribe el archivo SRT.
                with open(output_file, "w", encoding = "utf-8") as f:
                    f.write("1\n")
                    f.write(f"{start_time} --> {end_time}\n")
                    f.write(translated_text.strip() + "\n")
                    f.write("\n")
                return output_file
            except Exception as e:
                logging.error(f"Error al crear el archivo SRT: {e}")
                raise

            def process_video(video_path: str, target_lang: str = "es") -> Tuple[str, str]:
                """
                Procesamiento del video: extraccion de audio, transcripcion, traduccion y generacion de subtitulos.

                Args:
                    video_path (str): Ruta del video.
                    target_lang (str): Idioma al que se traducira el audio y texto.

                Returns:
                    Tuple[str, str]: Ruta del archivo de audio y ruta del archivo de subtitulos.

                Raises:
                    Exception: Si ocurre un error durante el procesamiento del video.
                """
                try:
                    if not os.path.exists(video_path):
                        raise FileNotFoundError("El archivo de video no existe.")
                    
                    # Extraer audio del video.
                    audio_path = extract_audio(video_path)
                    # Transcribir el audio extraido.
                    original_text = transcribe_audio(audio_path)
                    logging.info(f"Texto original detectado: {original_text}")
                    # Traducir el texto trnscrito al idioma de destino.
                    translated_text = translate_text(original_text, target_lang)
                    logging.info(f"Texto traducido: {translated_text}")

                    # Obtener la duracion real del video para sincronizar los subtitulos.
                    with VideoFileClip(video_path) as video:
                        video_duration = video.duration

                    # Generar el archivo de subtitulos en formato SRT.
                    srt_file = create_str_file(translated_text, video_duration)

                    # Generar un audio con el texto traducido.
                    audio_translated = generate_audio(translated_text, target_lang)
                    logging.info("Procesamiento completado.")
                    return srt_file, audio_translated
                except Exception as e:
                    logging.error(f"Error al procesar el video: {e}")
                    raise
            
            def main():
                """ Funcion principal que utiliza argparse para obtener los argumentos desde la linea de comandos.
                Ejemplo de uso:
                python traductor_video.py --video video.mp4 --lang en
                """
                parser = argparse.ArgumentParser(description = "Aplicacion para traducir audio de video y generar subtitulos.")
                parser.add_argument("--video", type = str, required = True, help = "Ruta del video a procesar.")
                parser.add_argument("--lang", type = str, default = "es", help = "Codigo del idioma de destino.")
                args = parser.parse_args()

                try:
                    srt_path, audio_path = process_video(args.video, args.lang)
                    logging.info(f"Subtitulos generados: {srt_path}")
                    logging.info(f"Audio traducido: {audio_path}")
                except Exception as e:
                    logging.error("No se pudo completar el procesamiento del video.")
                    sys.exit(1)

if __name__ == "__main__":
    main()