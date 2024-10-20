import struct
import os
import xml.etree.ElementTree as ET
import wave

def extract_xml_from_pdk(pdk_path):
    try:
        with open(pdk_path, 'rb') as f:
            content = f.read()

        xml_start = content.find(b'<InstDef')
        xml_end = content.find(b'</InstDef>') + len(b'</InstDef>')

        if xml_start == -1 or xml_end == -1:
            print("Error: Could not find XML portion in the file.")
            return None

        xml_data = content[xml_start:xml_end].decode('utf-8', errors='ignore')
        return xml_data
    except Exception as e:
        print(f"Error extracting XML from PDK file: {e}")
        return None

def parse_pdk_xml(xml_data):
    try:
        tree = ET.ElementTree(ET.fromstring(xml_data))
        sounds = []
        instrument_name = ""

        instrument = tree.find(".//Instrument")
        if instrument is not None:
            instrument_name = instrument.attrib.get('Name', 'Unknown')

        # Extract sound data from XML
        for sound in tree.findall(".//Sound"):
            sounds.append({
                'group': int(sound.attrib['Group']),
                'chan': int(sound.attrib['Chan']),
                'bps': int(sound.attrib['BPS']),
                'rate': int(sound.attrib['Rate']),
                'nSmp': int(sound.attrib['nSmp']),
                'offset': int(sound.attrib['Offset']),
                'offset1': int(sound.attrib['Offset1']),  # Optional second offset
                'vol': float(sound.attrib['Vol']),
                'maxComp': float(sound.attrib['MaxComp']),
            })

        print(f"Extracted {len(sounds)} sound entries from the XML for instrument '{instrument_name}'.")
        return sounds, instrument_name

    except Exception as e:
        print(f"Error parsing the XML data: {e}")
        return [], ""

def extract_audio(pdk_path, sound_info):
    try:
        with open(pdk_path, 'rb') as f:
            f.seek(sound_info['offset'])
            sample_data = f.read(sound_info['nSmp'] * (sound_info['bps'] // 8))
            return sample_data

    except Exception as e:
        print(f"Error extracting audio from the PDK file: {e}")
        return None

def save_as_wav(folder, filename, audio_data, channels, sample_width, sample_rate):
    try:
        filepath = os.path.join(folder, filename)
        
        with wave.open(filepath, 'wb') as wf:
            wf.setnchannels(channels)
            wf.setsampwidth(sample_width)
            wf.setframerate(sample_rate)
            wf.writeframes(audio_data)

        print(f"Saved WAV file to {filepath}")
    except Exception as e:
        print(f"Error saving WAV file: {e}")

def create_output_folder(folder):
    try:
        if not os.path.exists(folder):
            os.makedirs(folder)
        print(f"Output folder: {folder}")
    except Exception as e:
        print(f"Error creating output folder: {e}")

def extract_all_audio(pdk_path, output_folder):
    xml_data = extract_xml_from_pdk(pdk_path)

    if xml_data is None:
        print("No valid XML data to parse.")
        return

    sounds, instrument_name = parse_pdk_xml(xml_data)

    if not sounds:
        print("No sounds to extract.")
        return

    create_output_folder(output_folder)

    for i, sound in enumerate(sounds):
        print(f"Processing sound {i+1} of {len(sounds)}...")
        audio_data = extract_audio(pdk_path, sound)

        if audio_data:
            wav_filename = f"{instrument_name}_sound_{i + 1}.wav"
            save_as_wav(
                output_folder, 
                wav_filename, 
                audio_data, 
                channels=sound['chan'], 
                sample_width=sound['bps'] // 8, 
                sample_rate=sound['rate']
            )
        else:
            print(f"Failed to extract audio for sound {i+1}.")

if __name__ == "__main__":
    try:
        pdk_path = 'MT-PowerDrumKit-Content.pdk'
        output_folder = 'extracted_sounds'

        if not os.path.exists(pdk_path):
            print(f"PDK file not found: {pdk_path}")
        else:
            print(f"PDK file found: {pdk_path}")

        extract_all_audio(pdk_path, output_folder)
    except Exception as e:
        print(f"Unexpected error: {e}")

    input("Press Enter to exit...")
