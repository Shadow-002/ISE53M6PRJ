# **LSB Steganography on Images**

## Abstract
**Steganography is the art of hiding information in text,images,audio,video or inside any other data, is gaining momentum as it scores over cryptography because it enables to embed the secret message to cover images. Steganographic techniques offer more promise in digital image processing and more difficult to detect. LSB technique suggests that data can be hidden in the least significant bits of the cover image and the human eye would be unable to notice the hidden image in the cover file. This technique can be used for hiding images in 24-bit rgb colored/8-bit/gray scale format. We emphasize strongly on image Steganography providing a strong focus on the LSB techniques in image Steganography. This paper explains the LSB embedding technique and presents the evaluation results for 1,2 Least significant bit for a .png file and a .bmp file.
Keywords : steganography, LSB**
## Introduction
The use of multimedia processing tools enables us to copy,modify and retransmit the information. In order to communicate secretly without involving others nor making anyone suspicious we can use steganography techniques. The difference between Steganography vs Cryptography is that Steganography is difficult to detect but Ciphertext is easy to detect. Many other security algorithms are being developed in order to encrypt the shared keys i.e Elliptic Curve Cryptography etc.
## Literature Survey

## High-Level Design Diagram


## Implementation
```python
from math import log10,sqrt
import cv2
import numpy as np
import types
%matplotlib inline
import matplotlib.pyplot as plt
from google.colab.patches import cv2_imshow

# if r[-3:-1] == 00: r[-1] = ~r[-1] ? return s
def Reversal_Transform(rgb_value):
    if rgb_value[-3:-1] == '00':
        if rgb_value[-1] == '0':
            updated_rgb_value = rgb_value[:-1] + '1'
        else:
            updated_rgb_value = rgb_value[:-1] + '0'
        return updated_rgb_value
    else:
        return rgb_value


# Use EX-OR on last 3 LSB bits
def xor_func(x):
	bin_fmt = bin(int(x[-2:]) ^ 3)
	updated_bin_fmt = x[:-2] + bin_fmt[2:]
	return updated_bin_fmt

def bin_to_Dec(y):
	rev_decimal = int(y,2)
	return rev_decimal

def enc_process(pixel_index):
	pixel0_binstr = msgToBin(pixel_index)
	updated_rgb_value = Reversal_Transform(pixel0_binstr)
	xored_value_str = xor_func(updated_rgb_value)
	updated_rgb_val2 = bin_to_Dec(xored_value_str)
	return updated_rgb_val2

def dcr_process(pixel_index):
	xored_value_str = xor_func(pixel_index)
	updated_rgb_value = Reversal_Transform(xored_value_str)
	#pixel0_binstr = msgToBin(pixel_index)
	#updated_rgb_val2 = bin_to_Dec(xored_value_str)
	return updated_rgb_value

def msgToBin(msg):
    if type(msg) == str:
        return ''.join([format(ord(i),"08b") for i in msg])
    elif type(msg) == bytes or type(msg) == np.ndarray:
        return [format(i,"08b") for i in msg]
    elif type(msg) == int or type(msg) == np.uint8:
        return format(msg,"08b")
    else:
        raise TypeError("I/P type not supported")

def hideData(img,secret_msg):
    nbytes = img.shape[0] * img.shape[1] * 3 // 8
    if len(secret_msg) > nbytes:
        raise ValueError("Secret message lenght is greater than img")
    secret_msg += "#####"
    data_index = 0
    bin_secret_msg = msgToBin(secret_msg)
    data_len = len(bin_secret_msg)
    for values in img:
        for pixel in values:
            r,g,b = msgToBin(pixel)
            if(data_index < data_len):
                pixel[0] = int(r[:-1]+bin_secret_msg[data_index],2)
                data_index += 1
                #####
                pixel[0] = enc_process(pixel[0])
                #####
            if(data_index < data_len):
                pixel[1] = int(g[:-1]+bin_secret_msg[data_index],2)
                data_index += 1
                #####
                pixel[1] = enc_process(pixel[1])
                #####
            if(data_index < data_len):
                pixel[2] = int(b[:-1]+bin_secret_msg[data_index],2)
                data_index += 1
                #####
                pixel[2] = enc_process(pixel[2])
                #####
            if data_index >= data_len:
                break
    cv2.imwrite('out.png',img)
    return img

def showData(img):
	bin_data = ""
	for values in img:
		for pixel in values:
			r,g,b = msgToBin(pixel)
			r_updated = dcr_process(r)
			bin_data += r_updated[-1]
			g_updated = dcr_process(g)
			bin_data += g_updated[-1]
			b_updated = dcr_process(b)
			bin_data += b_updated[-1]
	all_bytes = [ bin_data[i:i+8] for i in range(0,len(bin_data),8) ]
	decoded_data = ""
	for byte in all_bytes:
		decoded_data += chr(int(byte,2,))
		if decoded_data[-5:] == "#####":
			break;
	return decoded_data[:-5]

def encode_txt():
	img_name = input("Enter image name with extension : ")
	img = cv2.imread(img_name)
	resized_img = cv2.resize(img,(500,500))
	print("Original Image is as shown below : ")
	#cv2.imshow("Original Image",resized_img)
	plt.imshow(resized_img)
	plt.show()

	print("Original Image Shape : ",img.shape)

	data = input("Enter data to be encoded : ")
	if(len(data) == 0):
		raise ValueError('Data is empty')

	filename = input("Enter output filename with extension : ")
	encoded_img = hideData(img,data)
	# psnr = PSNR(img,encoded_img)
	# print("PSNR : ",psnr)
	cv2.imwrite(filename,encoded_img)

def decode_txt():
	img_name = input("Enter Coverfile name with extension : ")
	cover_img = cv2.imread(img_name)
	resized_img = cv2.resize(cover_img,(500,500))
	print("Cover Image is as shown below : ")
	#cv2.imshow("Cover Image",resized_img)
	plt.imshow(resized_img)
	plt.show()
	print("Cover Image Shape : ",cover_img.shape)

	txt = showData(cover_img)
	return txt

def UserInput():
	choice = int(input("1. Encode\n2. Decode\nChoice : "))
	if choice == 1:
		print("Encoding ..........")
		encode_txt()
		print("Encoded Successfully!!!")
	elif choice == 2:
		print("Decoding ..........")
		txt = decode_txt()
		print("Decoded Successfully!!! : ",txt)
	else:
		raise Exception("Incorrect Choice!!!")
```
## References
