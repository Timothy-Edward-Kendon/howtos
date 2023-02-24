# TexLive (Equinor PC - VPN)

1. Download install-tl.zip file from https://www.tug.org/texlive/acquire-netinstall.html.
2. Add the following lines to the top of `install-tl-windows.bat`:
   ```batch
    echo Add proxys:
    set http_proxy=http://www-proxy.statoil.no:80
    set https_proxy=http://www-proxy.statoil.no:80
    ```
3. Then run the `install-tl` script.