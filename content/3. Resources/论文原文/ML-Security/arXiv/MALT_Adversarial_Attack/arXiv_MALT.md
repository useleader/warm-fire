---
publish: true
---

Traceback (most recent call last):
  File "/home/yanzm/.cache/uv/archive-v0/yKJwtu7ozNQ-gnmvYB7bv/bin/markitdown", line 12, in <module>
    sys.exit(main())
             ^^^^^^
  File "/home/yanzm/.cache/uv/archive-v0/yKJwtu7ozNQ-gnmvYB7bv/lib/python3.11/site-packages/markitdown/__main__.py", line 196, in main
    result = markitdown.convert(
             ^^^^^^^^^^^^^^^^^^^
  File "/home/yanzm/.cache/uv/archive-v0/yKJwtu7ozNQ-gnmvYB7bv/lib/python3.11/site-packages/markitdown/_markitdown.py", line 283, in convert
    return self.convert_local(source, stream_info=stream_info, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/yanzm/.cache/uv/archive-v0/yKJwtu7ozNQ-gnmvYB7bv/lib/python3.11/site-packages/markitdown/_markitdown.py", line 337, in convert_local
    return self._convert(file_stream=fh, stream_info_guesses=guesses, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/yanzm/.cache/uv/archive-v0/yKJwtu7ozNQ-gnmvYB7bv/lib/python3.11/site-packages/markitdown/_markitdown.py", line 626, in _convert
    raise FileConversionException(attempts=failed_attempts)
markitdown._exceptions.FileConversionException: File conversion failed after 1 attempts:
 - PdfConverter threw PSEOF with message: Unexpected EOF

