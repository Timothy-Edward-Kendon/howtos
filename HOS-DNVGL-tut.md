1. log onto a standard RGS node (not a top node to the HPC)
2. make a directory and copy the zip file to the directory. 
3. Type
    ```bash
    unzip HOSM_workshop_20.zip
    cd HOSM_workshop_20
    python3 -m venv hosdnvgl
    source hosdnvgl/bin/activate.csh
    cd HOSM_workshop_20
    chmod 775 HOSM_3.8
    pip install spectral_wave_data-1.0.0rc2-py2.py3-none-linux_x86_64.whl
    pip install f90nml --proxy www-proxy.statoil.no
    pip install numpy --proxy www-proxy.statoil.no
    cd examples
    python ./write_and_run_inputfiles.py
    ```