
```python
#!/usr/bin/env python3
import os
import subprocess
import itertools
import sys
import glob
import time
import shutil

PNG_MAGIC_BYTES = b'\x89\x50\x4E\x47\x0D\x0A\x1A\x0A'
CHARACTER_SPACE = 'abcdefghijklmnopqrstuvwxyz'

def setup_loop_device(drive_image):
    print("Setting up loop device...")
    result = subprocess.run(['sudo', 'losetup', '-f', '--show', drive_image], 
                           capture_output=True, text=True)
    if result.returncode != 0:
        print(f"Error setting up loop device: {result.stderr}")
        sys.exit(1)
    loop_device = result.stdout.strip()
    print(f"Loop device set up at: {loop_device}")
    return loop_device

def try_bitlocker_keys(loop_device, keys_file, mount_dir):
    print(f"Trying BitLocker keys from {keys_file}...")
    os.makedirs(mount_dir, exist_ok=True)
    with open(keys_file, 'r') as f:
        keys = f.read().strip().split('\n')
    total_keys = len(keys)
    print(f"Found {total_keys} keys to try")
    for i, key in enumerate(keys):
        if i % 10 == 0:
            print(f"Trying key {i+1}/{total_keys}...")
        result = subprocess.run(['sudo', 'dislocker', '-r', '-V', loop_device, 
                                f'-p{key}', mount_dir], 
                                capture_output=True, text=True)
        if result.returncode == 0:
            print(f"\nSuccess! Found working BitLocker key: {key}")
            return key
    print("Failed to find a working BitLocker key")
    return None

def mount_dislocker_file(dislocker_dir, mount_point):
    print(f"Mounting decrypted volume to {mount_point}...")
    os.makedirs(mount_point, exist_ok=True)
    dislocker_file = os.path.join(dislocker_dir, "dislocker-file")
    result = subprocess.run(['sudo', 'mount', '-o', 'loop', dislocker_file, mount_point],
                           capture_output=True, text=True)
    if result.returncode != 0:
        print(f"Error mounting dislocker file: {result.stderr}")
        return False
    print(f"Successfully mounted decrypted volume at {mount_point}")
    return True

def find_encrypted_files(mount_point):
    encrypted_files = glob.glob(os.path.join(mount_point, "*.xxx.crypt"))
    print(f"Found {len(encrypted_files)} encrypted files")
    return encrypted_files

def xor_decrypt(data, password):
    password_bytes = password.encode()
    decrypted = bytearray()
    for i, byte in enumerate(data):
        decrypted.append(byte ^ password_bytes[i % len(password_bytes)])
    return decrypted

def find_xor_password(encrypted_file_path):
    print(f"Trying to find XOR password using {os.path.basename(encrypted_file_path)}...")
    with open(encrypted_file_path, 'rb') as f:
        encrypted_bytes = f.read(8)
    total_combinations = 26**4
    print(f"Testing {total_combinations} possible passwords...")
    start_time = time.time()
    count = 0
    for password_tuple in itertools.product(CHARACTER_SPACE, repeat=4):
        password = ''.join(password_tuple)
        count += 1
        if count % 100000 == 0:
            elapsed = time.time() - start_time
            progress = (count / total_combinations) * 100
            print(f"Progress: {progress:.2f}% ({count}/{total_combinations}), Elapsed: {elapsed:.2f}s")
        decrypted_bytes = xor_decrypt(encrypted_bytes, password)
        if decrypted_bytes == PNG_MAGIC_BYTES:
            print(f"\nFound XOR password: {password}")
            return password
    print("Could not find the correct XOR password")
    return None

def decrypt_and_save_files(encrypted_files, password, output_dir):
    print(f"\nDecrypting files with password: {password}")
    os.makedirs(output_dir, exist_ok=True)
    for file_path in encrypted_files:
        file_name = os.path.basename(file_path)
        print(f"Decrypting {file_name}...")
        with open(file_path, 'rb') as f:
            encrypted_data = f.read()
        decrypted_data = xor_decrypt(encrypted_data, password)
        if file_name.endswith('.xxx.crypt'):
            output_name = file_name[:-9]
        else:
            output_name = file_name + '.decrypted'
        output_path = os.path.join(output_dir, output_name)
        with open(output_path, 'wb') as f:
            f.write(decrypted_data)
        print(f"Saved decrypted file: {output_path}")

def cleanup(loop_device, mount_points):
    print("\nCleaning up...")
    for mount_point in mount_points:
        if os.path.ismount(mount_point):
            subprocess.run(['sudo', 'umount', mount_point], capture_output=True)
    subprocess.run(['sudo', 'losetup', '-d', loop_device], capture_output=True)
    print("Cleanup complete")

def main():
    drive_image = 'encrypted_usb.dd'
    keys_file = 'recovery_keys_dump.txt'
    dislocker_dir = './dislocker_mount'
    volume_mount = './bitlocker_mount'
    output_dir = './decrypted_files'
    if not os.path.exists(drive_image):
        print(f"Drive image not found: {drive_image}")
        return
    if not os.path.exists(keys_file):
        print(f"Keys file not found: {keys_file}")
        return
    loop_device = setup_loop_device(drive_image)
    try:
        bitlocker_key = try_bitlocker_keys(loop_device, keys_file, dislocker_dir)
        if not bitlocker_key:
            return
        if not mount_dislocker_file(dislocker_dir, volume_mount):
            return
        encrypted_files = find_encrypted_files(volume_mount)
        if not encrypted_files:
            print("No encrypted files found")
            return
        xor_password = find_xor_password(encrypted_files[0])
        if not xor_password:
            return
        decrypt_and_save_files(encrypted_files, xor_password, output_dir)
        print("\nRecovery complete! All files have been decrypted.")
        print(f"Decrypted files are in: {os.path.abspath(output_dir)}") 
    finally:
        cleanup(loop_device, [volume_mount, dislocker_dir])

if __name__ == "__main__":
    main()
```

