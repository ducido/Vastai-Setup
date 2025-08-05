# Vastai-Setup

# Set up ssh key to secure your connection to vastai server
##  Step 1: Check for Existing SSH Keys

```bash
ls -al ~/.ssh
```

If you see files like `id_rsa` or `id_ed25519`, you may already have a key. You can reuse it or create a new one.

---

### Step 2: Generate a New SSH Key

Use the following command (replace with your email):

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- Press `Enter` to save to default path (`~/.ssh/id_ed25519`) or change it any path you want
- Optionally enter a passphrase for extra security

---

### Step 3: Start the SSH Agent

```bash
eval "$(ssh-agent -s)"
```

Add your private key to the agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

---

### Step 4: Add the SSH Key to GitHub

Copy your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

1. Go to [https://cloud.vast.ai/manage-keys/](https://cloud.vast.ai/manage-keys/)
2. Paste the key into the **"SSH Keys"** field

---

# Script to connect to github via ssh and set up environment, download data, code, etc...
```bash
# Update latest ubuntu packages
apt-get update
apt-get install -y curl git-core

# Install and enable git lfs
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash && apt-get install -y git-lfs && git lfs install

# Set up ssh key to connect to github
mkdir -p ~/.ssh && chmod 700 ~/.ssh

cat > ~/.ssh/id_ed25519 <<- EOM
-----BEGIN OPENSSH PRIVATE KEY-----
-----END OPENSSH PRIVATE KEY-----
EOM

chmod 400 ~/.ssh/id_ed25519

ssh-keygen -F github.com || ssh-keyscan github.com >> ~/.ssh/known_hosts

git config --global user.email "nduc90313@gmail.com"
git config --global user.name "ducido"

# Download COCO dataset
# Download image zip files
wget http://images.cocodataset.org/zips/train2017.zip
wget http://images.cocodataset.org/zips/val2017.zip
wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip

# Create target directory
mkdir -p coco_2017
mkdir -p coco_2017/annotations

# Unzip into coco_dataset/
unzip train2017.zip -d coco_2017/
unzip val2017.zip -d coco_2017/
unzip annotations_trainval2017.zip -d coco_2017/

# Clone a repo
git clone https://github.com/longzw1997/Open-GroundingDino.git
cd Open-GroundingDino
pip install -r requirements.txt 
cd models/GroundingDINO/ops
python setup.py build install
python test.py
cd ../../..

```

## Add-on: Train GroundingDino on COCO datasets
### Convert data to ODVG format
```bash
cd Open-GroundingDino
python tools/coco2odvg.py --input /workspace/coco_2017/annotations/instances_train2017.json --output /workspace/coco_2017/annotations/coco2017_train_odvg.jsonl --idmap coco2017
```

Create config/mycoco_odvg.json with following content
```bash
{
  "train": [
    {
      "root": "/workspace/coco_2017/train2017/",
      "anno": "/workspace/coco_2017/annotations/coco2017_train_odvg.jsonl",
      "label_map": "./coco2017_label_map.json",
      "dataset_mode": "odvg"
    }
  ],
  "val": [
    {
      "root": "/workspace/coco_2017/val2017",
      "anno": "/workspace/coco_2017/instances_val2017.json",
      "label_map": null,
      "dataset_mode": "coco"
    }
  ]
}
```

### Train
```
bash train_dist.sh 1 config/cfg_coco.py config/mycoco_odvg.json ./logs
```



#### That's it!! Hope you guys doing well.
