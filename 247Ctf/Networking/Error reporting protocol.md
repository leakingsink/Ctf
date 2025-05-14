
```python
#!/usr/bin/env python3
import subprocess
import os
import tempfile

def extract_data_from_pcap():
    pcap_file = "error_reporting.pcap"  # Hardcoded path to the PCAP file
    print(f"[+] Processing PCAP file: {pcap_file}")
    with tempfile.NamedTemporaryFile(delete=False, suffix='.hex') as temp_hex_file, \
         tempfile.NamedTemporaryFile(delete=False, suffix='.bin') as temp_bin_file:
        hex_file_path = temp_hex_file.name
        bin_file_path = temp_bin_file.name
    print("[+] Extracting hex dump from PCAP using tshark...")
    tshark_cmd = f"tshark -r {pcap_file} -T fields -e data -Y 'icmp.type==0' 2>/dev/null"
    try:
        hex_data = subprocess.check_output(tshark_cmd, shell=True).decode('utf-8').strip()
        with open(hex_file_path, 'w') as f:
            for line in hex_data.split('\n'):
                if line.strip():  # Skip empty lines
                    f.write(line + '\n')
        print(f"[+] Hex dump extracted to: {hex_file_path}")
        print("[+] Converting hex dump to binary...")
        convert_cmd = f"xxd -r -p {hex_file_path} {bin_file_path}"
        subprocess.run(convert_cmd, shell=True, check=True)
        file_cmd = f"file {bin_file_path}"
        file_type = subprocess.check_output(file_cmd, shell=True).decode('utf-8').strip()
        print(f"[+] File type: {file_type}")
        output_file = "extracted_data"
        print(f"[+] Creating output file: {output_file}")
        subprocess.run(f"cp {bin_file_path} {output_file}", shell=True, check=True)
        print(f"[+] Data extraction complete! Output file: {output_file}")
        print("[+] You may need to add the appropriate file extension based on the file type.")
        os.unlink(hex_file_path)
        os.unlink(bin_file_path)
        
    except subprocess.CalledProcessError as e:
        print(f"[!] Error executing command: {e}")
    except Exception as e:
        print(f"[!] Error: {e}")
if __name__ == "__main__":
    extract_data_from_pcap()
```
