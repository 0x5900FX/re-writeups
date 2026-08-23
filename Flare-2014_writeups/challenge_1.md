Platform:
Windows

Flare-2014 -> Challenge1 


Approach:

Used PE Studio to analyze and know about the program


Used Dnsspy to turn that application to c# code. 

While looking into file. I stumbled upon Form1. When clicked it changes the image.
So i looked into it.

Found this code decoding the secret while scrambling data again.

```
private void btnDecode_Click(object sender, EventArgs e)
		{
			this.pbRoge.Image = Resources.bob_roge;
			byte[] dat_secret = Resources.dat_secret;
			string text = "";
			foreach (byte b in dat_secret)
			{
				text += (char)(((b >> 4) | (((int)b << 4) & 240)) ^ 41);
			}
			text += "\0";
			string text2 = "";
			for (int j = 0; j < text.Length; j += 2)
			{
				text2 += text[j + 1];
				text2 += text[j];
			}
			string text3 = "";
			for (int k = 0; k < text2.Length; k++)
			{
				char c = text2[k];
				text3 += (char)((byte)text2[k] ^ 102);
			}
			this.lbl_title.Text = text3;
		}
```

So i downloaded the data from resource it was named as dat_secret

So reading the code i tried to replicate it in python and thus able to sucessfully get the flag. The code is below.

```
with open('rev_challenge_1.dat_secret.encode', 'rb') as f:
    user_data = f.read()

print(user_data)
text = ''

for b in user_data:
    text += chr(((b >> 4 ) | ((b << 4) & 240)) ^ 41)
text += "\n"

print(text)

```
Got the Flag

3rmahg3rd.b0b.d0ge@flare-on.com