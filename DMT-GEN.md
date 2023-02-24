# DMT-GEN code generation

1. Download the repository

   ```sh
   git clone https://github.com/SINTEF/dmt-gen-py.git
   ```

2. Install the dependencies

   ```sh
   pip install dmtgen=0.0.2 --proxy www-proxy.statoil.no
   pip install dmtpy==0.0.2 --proxy www-proxy.statoil.no
   ```

3. Create a set of blueprints in SIMA
4. Run the code generator

   ```sh
   python .\generate.py --cleanup .\test\ForApp .\test\output
   ```
