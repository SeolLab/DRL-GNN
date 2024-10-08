name: Python package

on:
  push:
    branches: [ master ]

jobs:
  run:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python 3.9
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install Miniconda
      run: |
        wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
        bash miniconda.sh -b -p $HOME/miniconda
        echo "$HOME/miniconda/bin" >> $GITHUB_PATH
        echo "$HOME/miniconda/condabin" >> $GITHUB_PATH
        source $HOME/miniconda/bin/activate
        conda init bash
        conda create -n myenv python=3.9 -y
        echo "source $HOME/miniconda/bin/activate myenv" >> $GITHUB_ENV
    
    - name: Install dependencies
      run: |
        source $HOME/miniconda/bin/activate myenv
        pip install -r DRL-GNN/DQN/requirements.txt
        pip install -e DRL-GNN/DQN/gym-environments/
    
    - name: Set PYTHONPATH
      run: echo "PYTHONPATH=$(pwd)/DRL-GNN/DQN:${PYTHONPATH}" >> $GITHUB_ENV
    
    - name: Test with pytest
      run: |
        source $HOME/miniconda/bin/activate myenv
        python DRL-GNN/DQN/train_DQN.py
    
    - name: Evaluate model
      run: |
        source $HOME/miniconda/bin/activate myenv
        python DRL-GNN/DQN/evaluate_DQN.py -d ./Logs/expsample_DQN_agentLogs.txt
