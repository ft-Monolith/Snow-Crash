python -c "
s = bytearray(open('token', 'rb').read())
print(''.join(chr(b - i) for i, b in enumerate(s) if b - i > 0))
"
