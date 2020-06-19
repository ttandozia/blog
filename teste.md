{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "name": "DESCRITORES_COM_HOG.ipynb",
      "provenance": [],
      "toc_visible": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "accelerator": "GPU"
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "_23xJRO7Musm",
        "colab_type": "text"
      },
      "source": [
        "<a rel=\"license\" href=\"http://creativecommons.org/licenses/by-nc-sa/4.0/\"><img alt=\"Licença Creative Commons\" style=\"border-width:0\" src=\"https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png\" /></a><br /><span xmlns:dct=\"http://purl.org/dc/terms/\" href=\"http://purl.org/dc/dcmitype/Text\" property=\"dct:title\" rel=\"dct:type\">DESCRITORS_WITH_HOG</span> de <a xmlns:cc=\"http://creativecommons.org/ns#\" href=\"https://github.com/ttandozia/image_analysis\" property=\"cc:attributionName\" rel=\"cc:attributionURL\">ANDOZIA THIAGO TANURE</a> está licenciado com uma Licença <a rel=\"license\" href=\"http://creativecommons.org/licenses/by-nc-sa/4.0/\">Creative Commons - Atribuição-NãoComercial-CompartilhaIgual 4.0 Internacional</a>.<br />Baseado no trabalho disponível em <a xmlns:dct=\"http://purl.org/dc/terms/\" href=\"https://github.com/ttandozia/image_analysis/DESCRITORS_WITH_HOG.ipynb\" rel=\"dct:source\">https://github.com/ttandozia/image_analysis/DESCRITORS_WITH_HOG.ipynb</a>."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "SE0mNzYuS1bc",
        "colab_type": "text"
      },
      "source": [
        "# **PROCESSAMENTO DE IMAGEM**\n",
        "## **Identificação de diferentes tipos de fonte contidas em imagens binárias com uso de descritores extraídos separadamente e também com HOG (Histogram of Oriented Gradient).**\n",
        "## Utilização de modelos K-means e Random Forest para predição.\n",
        "\n",
        "#### **THIAGO TANURE ANDOZIA**\n",
        "\n",
        "**Jan/2020**"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "gjeLNBKyQfJf",
        "colab_type": "text"
      },
      "source": [
        "O desenvolvimento apresentado abaixo possui fins ligados à pesquisa acadêmica. Por estar usando o Google Colab, alguns códigos são específicos para conseguir extrair dados na plataforma.\n",
        "\n",
        "*   IMG011 = LETRA A\n",
        "*   IMG012 = LETRA B\n",
        "*   IMG013 = LETRA C"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "ypq2EgLVHO-w",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Para tirar o zip dos arquivos de treino\n",
        "!unzip train_set"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "tWcsfh8-JamG",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Importando os pacotes\n",
        "import os\n",
        "import cv2\n",
        "import pandas as pd\n",
        "import numpy as np\n",
        "import glob\n",
        "from matplotlib import pyplot as plt\n",
        "from google.colab import files\n",
        "from sklearn.preprocessing import StandardScaler\n",
        "from sklearn.cluster import KMeans\n",
        "from sklearn.ensemble import RandomForestClassifier\n",
        "from skimage.feature import hog\n",
        "from skimage import data, exposure\n",
        "\n",
        "#Metricas de avaliacao dos modelos\n",
        "from sklearn.metrics import classification_report, accuracy_score, f1_score, confusion_matrix\n",
        "\n",
        "%matplotlib inline\n",
        "plt.rcParams[\"figure.figsize\"] = [5,5]"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "NxLsRc6safq_",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando as bases para o treino e o teste\n",
        "x_treino = []\n",
        "y_treino = []\n",
        "\n",
        "x_teste = []\n",
        "y_teste = []\n",
        "\n",
        "# Criando as bases finais para envio\n",
        "final_treino = pd.DataFrame ()\n",
        "final_treino[\"IMAGEM\"] = ()\n",
        "final_treino[\"ANOTACAO\"] = ()\n",
        "\n",
        "final_teste = pd.DataFrame ()\n",
        "final_teste[\"IMAGEM\"] = ()\n",
        "final_teste[\"ANOTACAO\"] = ()"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "fJIWlnFo-oGd",
        "colab_type": "text"
      },
      "source": [
        "**BASE DE TREINO**"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "oUF52O9miueU",
        "colab_type": "code",
        "outputId": "146cc55a-14e6-4590-b243-81ce0b74f873",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 35
        }
      },
      "source": [
        "# Inserindo todas as imagens na base x e nomeando cada uma pelo nome da pasta na base y.\n",
        "# Inserindo também as informações na base final para envio\n",
        "fil = glob.glob (\"/content/*/*.png\")\n",
        "for myFile in fil:\n",
        "    image = cv2.imread (myFile, 0)\n",
        "    x_treino.append (image) # Agregando a imagem no x\n",
        "    y_treino.append (myFile[9:10]) # Agregando o label no y\n",
        "    final_treino.loc[myFile, 'IMAGEM'] = myFile[11:-4] # Agregando o nome da imagem no arquivo final\n",
        "    final_treino.loc[myFile, 'ANOTACAO'] = myFile[9:10] # Agregando o label da imagem no arquivo final\n",
        "\n",
        "print('x_treino shape:', np.array(x_treino).shape) # Analisando o tamando da base de treino"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "x_treino shape: (2397, 128, 128)\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "7wNGmfHrSdUR",
        "colab_type": "code",
        "outputId": "5863c865-1712-4bfd-c939-91d6d1b311ba",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Teste de verificação\n",
        "plt.imshow(x_treino[1500])\n",
        "print(y_treino[1500])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "C\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\nYQAAAABJRU5ErkJggg==\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "yYlOWNjBFX6-",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Resetando o index da base final\n",
        "final_treino = final_treino.reset_index(drop=True)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "b3qSqSmwV99d",
        "colab_type": "code",
        "outputId": "f44b3144-1198-4ac1-a4df-9ae87ae46f43",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 206
        }
      },
      "source": [
        "# Validação base treino final\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO\n",
              "0  img012-00236        B\n",
              "1  img012-00584        B\n",
              "2  img012-00406        B\n",
              "3  img012-00472        B\n",
              "4  img012-00554        B"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 7
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "QfOsmwEqkQcS",
        "colab_type": "text"
      },
      "source": [
        "**INICIANDO OS TRATAMENTOS NA BASE DE TREINO**"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "yFAgUyFgk9RB",
        "colab_type": "text"
      },
      "source": [
        "O primeiro tratamento que faremos é inverter as cores da base para que seja possível identificar as letras na cor branca e o fundo na cor preta.\n",
        "Para isso, utilziaremos o método \"NOT\" que irá fazer o negativo das imagens."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "hoaZSOvFlIX5",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando uma base para alocar as imagens alteradas\n",
        "x_treino_not = []"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "JdkS_lDWlIgV",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Alterando as cores da imagem e do fundo\n",
        "for image in x_treino:\n",
        "  img = cv2.bitwise_not(image)\n",
        "  x_treino_not.append(img)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Gd5XB_VSt8Kb",
        "colab_type": "code",
        "outputId": "1f34c795-021c-4084-d780-c83df5c009c0",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Teste de verificação\n",
        "plt.imshow(x_treino_not[1500])\n",
        "print(y_treino[1500])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "C\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "SaNEBsxtpsUe",
        "colab_type": "text"
      },
      "source": [
        "Aplicando o esqueleto"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "2DlhkKr-IiU1",
        "colab_type": "text"
      },
      "source": [
        "Agregaremos as definições vistas em aula para a criação do esqueleto de Zhang-Suen. Utilizei esse método por termos muitas fontes que são \"desenhadas\" e podem comprometer a estrutura se fizermos com o esqueleto visto anteriormente.\n",
        "\n",
        "Escolhi atuar com esqueletos para mitigar as variações nas espessuaras das diferentes fontes"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "xkm0b7RTpwD1",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "def neighbours(image, x, y):\n",
        "    \"Return 8-neighbours of image point P1(x,y), in a clockwise order\"\n",
        "    img = image\n",
        "    x_1, y_1, x1, y1 = x-1, y-1, x+1, y+1\n",
        "    \n",
        "    neigh = [img[x_1][y], img[x_1][y1], img[x][y1], img[x1][y1],\n",
        "             img[x1][y], img[x1][y_1], img[x][y_1], img[x_1][y_1]] \n",
        "    \n",
        "    return neigh   "
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "DhcSJv2fql9f",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "def transitions(neighbours):\n",
        "    \"No. of 0,1 patterns (transitions from 0 to 1) in the ordered sequence\"\n",
        "    n = neighbours + neighbours[0:1]\n",
        "    return sum( (n1, n2) == (0, 1) for n1, n2 in zip(n, n[1:]) )"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "fX5xdHehqmCE",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "def zhangSuen(image):\n",
        "    \"the Zhang-Suen Thinning Algorithm\"\n",
        "    skeleton = image.copy()  # deepcopy to protect the original image\n",
        "    skeleton = cv2.divide(skeleton, 255)\n",
        "    changing1 = changing2 = 1        #  the points to be removed (set as 0)\n",
        "    while changing1 or changing2:   #  iterates until no further changes occur in the image\n",
        "        # Step 1\n",
        "        changing1 = []\n",
        "        rows, columns = skeleton.shape               # x for rows, y for columns\n",
        "        for x in range(1, rows - 1):                     # No. of  rows\n",
        "            for y in range(1, columns - 1):            # No. of columns\n",
        "                P2, P3, P4, P5, P6, P7, P8, P9 = n = neighbours(skeleton, x, y)\n",
        "                if (skeleton[x][y] == 1     and    # Condition 0: Point P1 in the object regions \n",
        "                    2 <= sum(n) <= 6   and    # Condition 1: 2<= N(P1) <= 6\n",
        "                    transitions(n) == 1 and    # Condition 2: S(P1)=1  \n",
        "                    P2 * P4 * P6 == 0  and    # Condition 3   \n",
        "                    P4 * P6 * P8 == 0): # Condition 4\n",
        "                    changing1.append((x,y))\n",
        "        for x, y in changing1: \n",
        "            skeleton[x][y] = 0\n",
        "        # Step 2\n",
        "        changing2 = []\n",
        "        for x in range(1, rows - 1):\n",
        "            for y in range(1, columns - 1):\n",
        "                P2,P3,P4,P5,P6,P7,P8,P9 = n = neighbours(skeleton, x, y)\n",
        "                if (skeleton[x][y] == 1   and        # Condition 0\n",
        "                    2 <= sum(n) <= 6  and       # Condition 1\n",
        "                    transitions(n) == 1 and      # Condition 2\n",
        "                    P2 * P4 * P8 == 0 and       # Condition 3\n",
        "                    P2 * P6 * P8 == 0): # Condition 4\n",
        "                    changing2.append((x,y))    \n",
        "        for x, y in changing2: \n",
        "            skeleton[x][y] = 0\n",
        "\n",
        "    skeleton = cv2.multiply(skeleton, 255)\n",
        "    return skeleton"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "-O4n5sroF6Xb",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "#Base com esqueleto\n",
        "\n",
        "x_treino_esq = []\n",
        "count = 0\n",
        "\n",
        "for image in x_treino_not:\n",
        "  img_skel = image\n",
        "  _, img_skel = cv2.threshold(img_skel, 127, 255, cv2.THRESH_OTSU)\n",
        "  skeleton_new = zhangSuen(img_skel)\n",
        "  x_treino_esq.append(skeleton_new) \n",
        "  #cv2.imwrite('/content/ESQUELETO/'+str(final_treino.iloc[count][0])+'.png', skeleton_new) # Salvaria na pasta criada no bloco acima\n",
        "  print(count)\n",
        "  count += 1"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "26ZXCgSiU8yW",
        "colab_type": "code",
        "outputId": "2a878c76-2ab8-4a90-bd61-5f7f4f329da8",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Testando uma imagem com esqueleto\n",
        "plt.imshow(x_treino_esq[1500])\n",
        "print(y_treino[1500])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "C\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "tU2WH24mKLg1",
        "colab_type": "text"
      },
      "source": [
        "**INSERINDO OS DESCRITORES NA BASE DE TREINO**"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "dvvb5D1D5ziD",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a área das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "I2C6oWylYvJq",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as áreas\n",
        "base_area = []\n",
        "for img in x_treino_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  area = cv2.contourArea(biggest)\n",
        "  base_area.append(area)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "mhQYo7TCbXMq",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_area)).rename(columns={0:'AREA_OBJ'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "c_NJefST7_uC",
        "colab_type": "text"
      },
      "source": [
        "Aplicando Altura das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "hGY2Iqr48JJc",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as alturas\n",
        "base_altura = []\n",
        "for alt in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(alt)\n",
        "  altura = h\n",
        "  base_altura.append(altura)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "pknP8DTt7M0O",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de altura com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_altura)).rename(columns={0:'ALTURA'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "W1Ge1564nfBN",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a largura das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "k4qI5tOxniMR",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as larguras\n",
        "base_largura = []\n",
        "for lar in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(lar)\n",
        "  largura = w\n",
        "  base_largura.append(largura)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "nlOtcvVTbtse",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de largura com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_largura)).rename(columns={0:'LARGURA'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "HtPsp3yateyl",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a área do contorno"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "7FeFRqnOtio-",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as areas\n",
        "base_area2 = []\n",
        "for area in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(area)\n",
        "  objeto = w * h\n",
        "  base_area2.append(objeto)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "i9R-D7BXtd7d",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_area2)).rename(columns={0:'AREA_CONTORNO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "aH7qQRImpmuC",
        "colab_type": "text"
      },
      "source": [
        "Aplicando balanceamento horizontal"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "HzGPgQxMpzQB",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o balanceamento horizontal\n",
        "balan_hor = []\n",
        "for hor in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(hor)\n",
        "  horizontal = (64-(w/2))\n",
        "  balan_hor.append(horizontal)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "iK2Ylc39pqIP",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de balanceamento horizontal com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(balan_hor)).rename(columns={0:'BALAN_HOR'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "YsoivLewwFqn",
        "colab_type": "text"
      },
      "source": [
        "Aplicando balanceamento vertical"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "3gh4DEKWwH6x",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o balanceamento vertical\n",
        "balan_ver = []\n",
        "for ver in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(ver)\n",
        "  vertical = (64-(h/2))\n",
        "  balan_ver.append(vertical)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "k4DNxoQiuKFm",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de balanceamento vertical com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(balan_ver)).rename(columns={0:'BALAN_VER'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "0CVlBSvyw2Mm",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a razão das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "IrRbsl29w4pN",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as razões\n",
        "base_razao = []\n",
        "for raz in x_treino_esq:\n",
        "  x, y, w, h = cv2.boundingRect(raz)\n",
        "  razao = h/w\n",
        "  base_razao.append(razao)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "xQSzZbhuw4uU",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de razão com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_razao)).rename(columns={0:'RAZAO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "3pnmiU9B3lHo",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a compacidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "JzVsCbyK3nH_",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir a compacidade\n",
        "base_compacidade = []\n",
        "\n",
        "for img in x_treino_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  area = cv2.contourArea(biggest)\n",
        "  perimetro = cv2.arcLength(biggest, True)\n",
        "  compacidade = ((perimetro**2)/area) if area != 0 else 0\n",
        "  base_compacidade.append(compacidade)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "zKgt2r-lE3yk",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de compacidade com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_compacidade)).rename(columns={0:'COMPACIDADE'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "lo72GlysGTRk",
        "colab_type": "text"
      },
      "source": [
        "Aplicando o perímetro do contorno"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "_0xXTnZ5GVvd",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o perímetro do contorno\n",
        "base_perimetro = []\n",
        "\n",
        "for img in x_treino_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  perimetro = cv2.arcLength(biggest, True)\n",
        "  base_perimetro.append(perimetro)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "SHrfjv3UGkf-",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de perímetro com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_perimetro)).rename(columns={0:'PERIM_CONTORNO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "fzc1of3OG8JB",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a retangularidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Akad2f6R4HKD",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de área, largura e altura calculados na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_treino['RETANG.'] = final_treino['AREA_OBJ'] / (final_treino['ALTURA'] * final_treino['LARGURA'])"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "GKRSUak0GGIX",
        "colab_type": "text"
      },
      "source": [
        "Aplicando o número de Euler"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "_qoY0E31GIMj",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o número de Euler\n",
        "base_euler = []\n",
        "\n",
        "for img in x_treino_esq:\n",
        "  contornos,_ = cv2.findContours(img,cv2.RETR_CCOMP,cv2.CHAIN_APPROX_SIMPLE)\n",
        "  euler = 2 - len(contornos) # Tiramos 2 do contorno pois 1 representa a letra em si e o outro 1 é da fórmula.\n",
        "  base_euler.append(euler)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "uyacyiZaGIQE",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de Euler com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_euler)).rename(columns={0:'EULER'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "qZZYRwYh8odM",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a área e perimetro do fecho convexo"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "gF97DbHT8qly",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as áreas\n",
        "base_area3 = []\n",
        "base_perimetro2 = []\n",
        "\n",
        "for img in x_treino_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  hull = cv2.convexHull(biggest)\n",
        "  area = cv2.contourArea(hull)\n",
        "  base_area3.append(area)\n",
        "  perimetro = cv2.arcLength(hull, True)\n",
        "  base_perimetro2.append(perimetro)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "RGl0nkYBEkAy",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área e perímetro do fecho com as já existentes\n",
        "final_treino = final_treino.join(pd.DataFrame(base_area3)).rename(columns={0:'AREA_FECHO'})\n",
        "final_treino = final_treino.join(pd.DataFrame(base_perimetro2)).rename(columns={0:'PERIM_FECHO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "PMsa0JbeMsUZ",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a convexidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "r_igwcn2Mu5k",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de perímetro do fecho e do contorno na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_treino['CONVEXIDADE'] = final_treino['PERIM_FECHO'] / final_treino['PERIM_CONTORNO']"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "VS-gtxJKIl3G",
        "colab_type": "text"
      },
      "source": [
        "Aplicando a solidez"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "V0wKvU2DInPp",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de área do fecho e do contorno na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_treino['SOLIDEZ'] = final_treino['AREA_CONTORNO'] / final_treino['AREA_FECHO']"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "1060c859-be7b-4ace-e6b0-bf6c0d69bfe8",
        "id": "KXF6GLdQEQWG",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "      <td>3284.0</td>\n",
              "      <td>89</td>\n",
              "      <td>63</td>\n",
              "      <td>5607</td>\n",
              "      <td>32.5</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.412698</td>\n",
              "      <td>25.311170</td>\n",
              "      <td>288.308656</td>\n",
              "      <td>0.585696</td>\n",
              "      <td>-1</td>\n",
              "      <td>3929.5</td>\n",
              "      <td>256.980317</td>\n",
              "      <td>0.891338</td>\n",
              "      <td>1.426899</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "      <td>3826.0</td>\n",
              "      <td>81</td>\n",
              "      <td>57</td>\n",
              "      <td>4617</td>\n",
              "      <td>35.5</td>\n",
              "      <td>23.5</td>\n",
              "      <td>1.421053</td>\n",
              "      <td>17.699231</td>\n",
              "      <td>260.225396</td>\n",
              "      <td>0.828677</td>\n",
              "      <td>-1</td>\n",
              "      <td>4054.0</td>\n",
              "      <td>246.749827</td>\n",
              "      <td>0.948216</td>\n",
              "      <td>1.138875</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "      <td>3612.0</td>\n",
              "      <td>93</td>\n",
              "      <td>64</td>\n",
              "      <td>5952</td>\n",
              "      <td>32.0</td>\n",
              "      <td>17.5</td>\n",
              "      <td>1.453125</td>\n",
              "      <td>32.847826</td>\n",
              "      <td>344.450791</td>\n",
              "      <td>0.606855</td>\n",
              "      <td>-1</td>\n",
              "      <td>5469.0</td>\n",
              "      <td>289.761270</td>\n",
              "      <td>0.841227</td>\n",
              "      <td>1.088316</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  PERIM_FECHO  CONVEXIDADE   SOLIDEZ\n",
              "0  img012-00236        B    4358.5  ...   282.962375     0.909152  1.373701\n",
              "1  img012-00584        B    3284.0  ...   256.980317     0.891338  1.426899\n",
              "2  img012-00406        B    3826.0  ...   246.749827     0.948216  1.138875\n",
              "3  img012-00472        B    4358.5  ...   282.962375     0.909152  1.373701\n",
              "4  img012-00554        B    3612.0  ...   289.761270     0.841227  1.088316\n",
              "\n",
              "[5 rows x 17 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 57
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "695wdcrGPhg2",
        "colab_type": "text"
      },
      "source": [
        "Nrmalizando a base"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "D8FFwu4VPyOm",
        "colab_type": "code",
        "outputId": "1ce2d8f0-2820-4cec-d1ca-3f541a774c5d",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 35
        }
      },
      "source": [
        "# Criando a escala\n",
        "\n",
        "scaler = StandardScaler()\n",
        "scaler.fit(final_treino.iloc[:,3:17])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "StandardScaler(copy=True, with_mean=True, with_std=True)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 58
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "5PGrYPWGOzO5",
        "colab_type": "code",
        "outputId": "ff2a9828-c19f-4612-a725-b5517e880c14",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 206
        }
      },
      "source": [
        "# Escalando a base de treino\n",
        "\n",
        "final_treino_scaled = pd.DataFrame(scaler.transform(final_treino.iloc[:,3:17]))\n",
        "final_treino_scaled.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "      <th>3</th>\n",
              "      <th>4</th>\n",
              "      <th>5</th>\n",
              "      <th>6</th>\n",
              "      <th>7</th>\n",
              "      <th>8</th>\n",
              "      <th>9</th>\n",
              "      <th>10</th>\n",
              "      <th>11</th>\n",
              "      <th>12</th>\n",
              "      <th>13</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>0.303837</td>\n",
              "      <td>0.439304</td>\n",
              "      <td>0.576152</td>\n",
              "      <td>-0.439304</td>\n",
              "      <td>-0.303837</td>\n",
              "      <td>-0.339693</td>\n",
              "      <td>-0.341409</td>\n",
              "      <td>-0.740824</td>\n",
              "      <td>1.292940</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>0.820618</td>\n",
              "      <td>0.849431</td>\n",
              "      <td>1.255755</td>\n",
              "      <td>-0.032766</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>0.303837</td>\n",
              "      <td>-0.649192</td>\n",
              "      <td>-0.490560</td>\n",
              "      <td>0.649192</td>\n",
              "      <td>-0.303837</td>\n",
              "      <td>0.377600</td>\n",
              "      <td>-0.341098</td>\n",
              "      <td>-1.012867</td>\n",
              "      <td>1.141268</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>-0.262385</td>\n",
              "      <td>-0.247545</td>\n",
              "      <td>1.129855</td>\n",
              "      <td>-0.028727</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>-0.755505</td>\n",
              "      <td>-1.084591</td>\n",
              "      <td>-1.281604</td>\n",
              "      <td>1.084591</td>\n",
              "      <td>0.755505</td>\n",
              "      <td>0.399657</td>\n",
              "      <td>-0.341866</td>\n",
              "      <td>-1.346063</td>\n",
              "      <td>2.015631</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>-0.142426</td>\n",
              "      <td>-0.679482</td>\n",
              "      <td>1.531824</td>\n",
              "      <td>-0.050592</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>0.303837</td>\n",
              "      <td>0.439304</td>\n",
              "      <td>0.576152</td>\n",
              "      <td>-0.439304</td>\n",
              "      <td>-0.303837</td>\n",
              "      <td>-0.339693</td>\n",
              "      <td>-0.341409</td>\n",
              "      <td>-0.740824</td>\n",
              "      <td>1.292940</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>0.820618</td>\n",
              "      <td>0.849431</td>\n",
              "      <td>1.255755</td>\n",
              "      <td>-0.032766</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>0.833509</td>\n",
              "      <td>-0.576626</td>\n",
              "      <td>-0.214893</td>\n",
              "      <td>0.576626</td>\n",
              "      <td>-0.833509</td>\n",
              "      <td>0.484337</td>\n",
              "      <td>-0.340337</td>\n",
              "      <td>-0.346762</td>\n",
              "      <td>1.217407</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>1.220963</td>\n",
              "      <td>1.136484</td>\n",
              "      <td>0.775714</td>\n",
              "      <td>-0.054430</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         0         1         2   ...        11        12        13\n",
              "0  0.303837  0.439304  0.576152  ...  0.849431  1.255755 -0.032766\n",
              "1  0.303837 -0.649192 -0.490560  ... -0.247545  1.129855 -0.028727\n",
              "2 -0.755505 -1.084591 -1.281604  ... -0.679482  1.531824 -0.050592\n",
              "3  0.303837  0.439304  0.576152  ...  0.849431  1.255755 -0.032766\n",
              "4  0.833509 -0.576626 -0.214893  ...  1.136484  0.775714 -0.054430\n",
              "\n",
              "[5 rows x 14 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 59
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "QXXWozItVPYl",
        "colab_type": "text"
      },
      "source": [
        "Aplicando o K-means no treino"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "RkSMHe44Vu0k",
        "colab_type": "code",
        "outputId": "dcd2d05d-b87a-45fb-e22d-e32074b71deb",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 72
        }
      },
      "source": [
        "# Criando o modelo e dando fit na base escalada\n",
        "\n",
        "kmeans = KMeans(n_clusters=3, init='k-means++', max_iter=300, n_init=10, random_state=0)\n",
        "kmeans.fit(final_treino_scaled)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "KMeans(algorithm='auto', copy_x=True, init='k-means++', max_iter=300,\n",
              "       n_clusters=3, n_init=10, n_jobs=None, precompute_distances='auto',\n",
              "       random_state=0, tol=0.0001, verbose=0)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 60
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "BvFaYp_YVRQw",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Rodando o predict e já agregando na base final\n",
        "\n",
        "final_treino['pred_y_kmeans'] = kmeans.predict(final_treino_scaled)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Um57bCYXVOXb",
        "colab_type": "code",
        "outputId": "d6226caa-1562-4aaf-cd61-562f154e4218",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "      <td>3284.0</td>\n",
              "      <td>89</td>\n",
              "      <td>63</td>\n",
              "      <td>5607</td>\n",
              "      <td>32.5</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.412698</td>\n",
              "      <td>25.311170</td>\n",
              "      <td>288.308656</td>\n",
              "      <td>0.585696</td>\n",
              "      <td>-1</td>\n",
              "      <td>3929.5</td>\n",
              "      <td>256.980317</td>\n",
              "      <td>0.891338</td>\n",
              "      <td>1.426899</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "      <td>3826.0</td>\n",
              "      <td>81</td>\n",
              "      <td>57</td>\n",
              "      <td>4617</td>\n",
              "      <td>35.5</td>\n",
              "      <td>23.5</td>\n",
              "      <td>1.421053</td>\n",
              "      <td>17.699231</td>\n",
              "      <td>260.225396</td>\n",
              "      <td>0.828677</td>\n",
              "      <td>-1</td>\n",
              "      <td>4054.0</td>\n",
              "      <td>246.749827</td>\n",
              "      <td>0.948216</td>\n",
              "      <td>1.138875</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "      <td>3612.0</td>\n",
              "      <td>93</td>\n",
              "      <td>64</td>\n",
              "      <td>5952</td>\n",
              "      <td>32.0</td>\n",
              "      <td>17.5</td>\n",
              "      <td>1.453125</td>\n",
              "      <td>32.847826</td>\n",
              "      <td>344.450791</td>\n",
              "      <td>0.606855</td>\n",
              "      <td>-1</td>\n",
              "      <td>5469.0</td>\n",
              "      <td>289.761270</td>\n",
              "      <td>0.841227</td>\n",
              "      <td>1.088316</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  CONVEXIDADE   SOLIDEZ  pred_y_kmeans\n",
              "0  img012-00236        B    4358.5  ...     0.909152  1.373701              0\n",
              "1  img012-00584        B    3284.0  ...     0.891338  1.426899              2\n",
              "2  img012-00406        B    3826.0  ...     0.948216  1.138875              2\n",
              "3  img012-00472        B    4358.5  ...     0.909152  1.373701              0\n",
              "4  img012-00554        B    3612.0  ...     0.841227  1.088316              0\n",
              "\n",
              "[5 rows x 18 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 62
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "5T1CxxbnYJYa",
        "colab_type": "text"
      },
      "source": [
        "Aplicando Random Forest no treino"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "cMrieSnLZNIA",
        "colab_type": "code",
        "outputId": "b4dfd8b2-bdf1-4a20-b61c-581d45b71355",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 203
        }
      },
      "source": [
        "# Criando o modelo de RF e dando fit na base escalada com o rótulo sendo a coluna \"Anotação\" da base final\n",
        "\n",
        "model_rf = RandomForestClassifier()\n",
        "model_rf.fit(final_treino_scaled, final_treino['ANOTACAO'])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "/usr/local/lib/python3.6/dist-packages/sklearn/ensemble/forest.py:245: FutureWarning: The default value of n_estimators will change from 10 in version 0.20 to 100 in 0.22.\n",
            "  \"10 in version 0.20 to 100 in 0.22.\", FutureWarning)\n"
          ],
          "name": "stderr"
        },
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "RandomForestClassifier(bootstrap=True, class_weight=None, criterion='gini',\n",
              "                       max_depth=None, max_features='auto', max_leaf_nodes=None,\n",
              "                       min_impurity_decrease=0.0, min_impurity_split=None,\n",
              "                       min_samples_leaf=1, min_samples_split=2,\n",
              "                       min_weight_fraction_leaf=0.0, n_estimators=10,\n",
              "                       n_jobs=None, oob_score=False, random_state=None,\n",
              "                       verbose=0, warm_start=False)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 63
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "IBAcW6jhYMJf",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Predict na base e já agregando a coluna ao final\n",
        "\n",
        "final_treino['pred_y_rf'] = model_rf.predict(final_treino_scaled)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "HPyv3s5xOzbZ",
        "colab_type": "code",
        "outputId": "5d5c2219-0918-4dad-f944-3502228f5ae9",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "      <td>3284.0</td>\n",
              "      <td>89</td>\n",
              "      <td>63</td>\n",
              "      <td>5607</td>\n",
              "      <td>32.5</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.412698</td>\n",
              "      <td>25.311170</td>\n",
              "      <td>288.308656</td>\n",
              "      <td>0.585696</td>\n",
              "      <td>-1</td>\n",
              "      <td>3929.5</td>\n",
              "      <td>256.980317</td>\n",
              "      <td>0.891338</td>\n",
              "      <td>1.426899</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "      <td>3826.0</td>\n",
              "      <td>81</td>\n",
              "      <td>57</td>\n",
              "      <td>4617</td>\n",
              "      <td>35.5</td>\n",
              "      <td>23.5</td>\n",
              "      <td>1.421053</td>\n",
              "      <td>17.699231</td>\n",
              "      <td>260.225396</td>\n",
              "      <td>0.828677</td>\n",
              "      <td>-1</td>\n",
              "      <td>4054.0</td>\n",
              "      <td>246.749827</td>\n",
              "      <td>0.948216</td>\n",
              "      <td>1.138875</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "      <td>3612.0</td>\n",
              "      <td>93</td>\n",
              "      <td>64</td>\n",
              "      <td>5952</td>\n",
              "      <td>32.0</td>\n",
              "      <td>17.5</td>\n",
              "      <td>1.453125</td>\n",
              "      <td>32.847826</td>\n",
              "      <td>344.450791</td>\n",
              "      <td>0.606855</td>\n",
              "      <td>-1</td>\n",
              "      <td>5469.0</td>\n",
              "      <td>289.761270</td>\n",
              "      <td>0.841227</td>\n",
              "      <td>1.088316</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...   SOLIDEZ  pred_y_kmeans  pred_y_rf\n",
              "0  img012-00236        B    4358.5  ...  1.373701              0          B\n",
              "1  img012-00584        B    3284.0  ...  1.426899              2          B\n",
              "2  img012-00406        B    3826.0  ...  1.138875              2          B\n",
              "3  img012-00472        B    4358.5  ...  1.373701              0          B\n",
              "4  img012-00554        B    3612.0  ...  1.088316              0          B\n",
              "\n",
              "[5 rows x 19 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 65
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "pmyxo6CUgy_4",
        "colab_type": "code",
        "outputId": "fd3fdb75-73df-41ef-fb99-e39073d515e9",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo Kmeans\n",
        "\n",
        "pd.crosstab(final_treino[\"ANOTACAO\"],final_treino[\"pred_y_kmeans\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>A</th>\n",
              "      <td>62.327910</td>\n",
              "      <td>13.016270</td>\n",
              "      <td>24.655820</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>B</th>\n",
              "      <td>45.556946</td>\n",
              "      <td>6.257822</td>\n",
              "      <td>48.185232</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>C</th>\n",
              "      <td>63.829787</td>\n",
              "      <td>4.881101</td>\n",
              "      <td>31.289111</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_kmeans          0          1          2\n",
              "ANOTACAO                                      \n",
              "A              62.327910  13.016270  24.655820\n",
              "B              45.556946   6.257822  48.185232\n",
              "C              63.829787   4.881101  31.289111"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 66
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "DWrzrToLiNev",
        "colab_type": "code",
        "outputId": "1ba7b218-f3f1-42aa-eb9a-402c08e1ec10",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e a Random Forest\n",
        "pd.crosstab(final_treino[\"ANOTACAO\"],final_treino[\"pred_y_rf\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>A</th>\n",
              "      <th>B</th>\n",
              "      <th>C</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>A</th>\n",
              "      <td>99.874844</td>\n",
              "      <td>0.000000</td>\n",
              "      <td>0.125156</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>B</th>\n",
              "      <td>0.125156</td>\n",
              "      <td>99.874844</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>C</th>\n",
              "      <td>0.000000</td>\n",
              "      <td>0.000000</td>\n",
              "      <td>100.000000</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_rf          A          B           C\n",
              "ANOTACAO                                   \n",
              "A          99.874844   0.000000    0.125156\n",
              "B           0.125156  99.874844    0.000000\n",
              "C           0.000000   0.000000  100.000000"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 67
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "V2zOwzNMmazs",
        "colab_type": "text"
      },
      "source": [
        "Aqui já podemos analisar que o modelo de árvores performou muito superior ao K-means com os parâmetros default. Vamos aplicar na base de testes e também comparar com o HOG para analisar e comparar as performances."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "LXg3HB3T7K5Y",
        "colab_type": "text"
      },
      "source": [
        "Criando a base HOG (treino) e aplicando o K-means e random forest na mesma"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "K7n2K3ajmrAB",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# HOG\n",
        "base_hog = []\n",
        "\n",
        "for img in x_treino_esq:\n",
        "  fd, hog_image = hog(img, orientations=8, pixels_per_cell=(16, 16),\n",
        "                    cells_per_block=(1, 1), visualize=True, multichannel=False)\n",
        "  base_hog.append(hog_image)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Ie30BgFYrzCP",
        "colab_type": "code",
        "outputId": "976e4104-469a-41a2-c1f1-7fca3a8f79c0",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Visualizando uma imagem com HOG\n",
        "\n",
        "plt.imshow(base_hog[1500])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "<matplotlib.image.AxesImage at 0x7f4a4a7e0438>"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 70
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\nnIgE6/8BCcxhv7IpXvUAAAAASUVORK5CYII=\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "NojcM_pV5d7t",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Ajustando as dimensões da base para aplicação do k-means\n",
        "\n",
        "base_hog_ok = np.array(base_hog)\n",
        "nsamples, nx, ny = base_hog_ok.shape\n",
        "base_hog_final = base_hog_ok.reshape((nsamples,nx*ny))"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Her0jbCGsXey",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Construindo os modelos na base HOG\n",
        "\n",
        "# K-means\n",
        "\n",
        "kmeans_hog = KMeans(n_clusters=3, init='k-means++', max_iter=300, n_init=10, random_state=0)\n",
        "kmeans_hog.fit(base_hog_final)\n",
        "\n",
        "final_treino['pred_y_kmeans_hog'] = kmeans_hog.predict(base_hog_final)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "ikcb7V-R6AZm",
        "colab_type": "code",
        "outputId": "fbda4491-d358-46ce-8d0d-7b016f00953c",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "      <td>3284.0</td>\n",
              "      <td>89</td>\n",
              "      <td>63</td>\n",
              "      <td>5607</td>\n",
              "      <td>32.5</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.412698</td>\n",
              "      <td>25.311170</td>\n",
              "      <td>288.308656</td>\n",
              "      <td>0.585696</td>\n",
              "      <td>-1</td>\n",
              "      <td>3929.5</td>\n",
              "      <td>256.980317</td>\n",
              "      <td>0.891338</td>\n",
              "      <td>1.426899</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "      <td>3826.0</td>\n",
              "      <td>81</td>\n",
              "      <td>57</td>\n",
              "      <td>4617</td>\n",
              "      <td>35.5</td>\n",
              "      <td>23.5</td>\n",
              "      <td>1.421053</td>\n",
              "      <td>17.699231</td>\n",
              "      <td>260.225396</td>\n",
              "      <td>0.828677</td>\n",
              "      <td>-1</td>\n",
              "      <td>4054.0</td>\n",
              "      <td>246.749827</td>\n",
              "      <td>0.948216</td>\n",
              "      <td>1.138875</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "      <td>3612.0</td>\n",
              "      <td>93</td>\n",
              "      <td>64</td>\n",
              "      <td>5952</td>\n",
              "      <td>32.0</td>\n",
              "      <td>17.5</td>\n",
              "      <td>1.453125</td>\n",
              "      <td>32.847826</td>\n",
              "      <td>344.450791</td>\n",
              "      <td>0.606855</td>\n",
              "      <td>-1</td>\n",
              "      <td>5469.0</td>\n",
              "      <td>289.761270</td>\n",
              "      <td>0.841227</td>\n",
              "      <td>1.088316</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  pred_y_kmeans  pred_y_rf  pred_y_kmeans_hog\n",
              "0  img012-00236        B    4358.5  ...              0          B                  2\n",
              "1  img012-00584        B    3284.0  ...              2          B                  2\n",
              "2  img012-00406        B    3826.0  ...              2          B                  2\n",
              "3  img012-00472        B    4358.5  ...              0          B                  2\n",
              "4  img012-00554        B    3612.0  ...              0          B                  2\n",
              "\n",
              "[5 rows x 20 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 73
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "6iUesk1561HM",
        "colab_type": "code",
        "outputId": "de5618bc-0a23-4611-99da-7097b9b34177",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo Kmeans_hog\n",
        "\n",
        "pd.crosstab(final_treino[\"ANOTACAO\"],final_treino[\"pred_y_kmeans_hog\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>A</th>\n",
              "      <td>98.748436</td>\n",
              "      <td>1.251564</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>B</th>\n",
              "      <td>13.391740</td>\n",
              "      <td>1.627034</td>\n",
              "      <td>84.981227</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>C</th>\n",
              "      <td>6.758448</td>\n",
              "      <td>92.740926</td>\n",
              "      <td>0.500626</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_kmeans_hog          0          1          2\n",
              "ANOTACAO                                          \n",
              "A                  98.748436   1.251564   0.000000\n",
              "B                  13.391740   1.627034  84.981227\n",
              "C                   6.758448  92.740926   0.500626"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 74
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "XGEnt4jy8RNv",
        "colab_type": "text"
      },
      "source": [
        "Como podemos notar, quando aplicamos o HOG o k-means melhora consideravelmente sua assertividade, quando comparado com a aplicação feita nos descritores anteriores."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "sVn0MgNn8dKy",
        "colab_type": "code",
        "outputId": "73962161-2a16-4431-f108-260627e99eb1",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 203
        }
      },
      "source": [
        "# Criando o modelo de RF e dando fit na base HOG com o rótulo sendo a coluna \"Anotação\" da base final\n",
        "\n",
        "model_rf_hog = RandomForestClassifier()\n",
        "model_rf_hog.fit(base_hog_final, final_treino['ANOTACAO'])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "/usr/local/lib/python3.6/dist-packages/sklearn/ensemble/forest.py:245: FutureWarning: The default value of n_estimators will change from 10 in version 0.20 to 100 in 0.22.\n",
            "  \"10 in version 0.20 to 100 in 0.22.\", FutureWarning)\n"
          ],
          "name": "stderr"
        },
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "RandomForestClassifier(bootstrap=True, class_weight=None, criterion='gini',\n",
              "                       max_depth=None, max_features='auto', max_leaf_nodes=None,\n",
              "                       min_impurity_decrease=0.0, min_impurity_split=None,\n",
              "                       min_samples_leaf=1, min_samples_split=2,\n",
              "                       min_weight_fraction_leaf=0.0, n_estimators=10,\n",
              "                       n_jobs=None, oob_score=False, random_state=None,\n",
              "                       verbose=0, warm_start=False)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 75
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "i3G54EI_6_Va",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Predict na base HOG e já agregando a coluna ao final\n",
        "\n",
        "final_treino['pred_y_rf_hog'] = model_rf_hog.predict(base_hog_final)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "E5W3S_5b83Xy",
        "colab_type": "code",
        "outputId": "b0e453f9-aed2-404f-add6-79d006d193a0",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_treino.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "      <th>pred_y_rf_hog</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img012-00236</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img012-00584</td>\n",
              "      <td>B</td>\n",
              "      <td>3284.0</td>\n",
              "      <td>89</td>\n",
              "      <td>63</td>\n",
              "      <td>5607</td>\n",
              "      <td>32.5</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.412698</td>\n",
              "      <td>25.311170</td>\n",
              "      <td>288.308656</td>\n",
              "      <td>0.585696</td>\n",
              "      <td>-1</td>\n",
              "      <td>3929.5</td>\n",
              "      <td>256.980317</td>\n",
              "      <td>0.891338</td>\n",
              "      <td>1.426899</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img012-00406</td>\n",
              "      <td>B</td>\n",
              "      <td>3826.0</td>\n",
              "      <td>81</td>\n",
              "      <td>57</td>\n",
              "      <td>4617</td>\n",
              "      <td>35.5</td>\n",
              "      <td>23.5</td>\n",
              "      <td>1.421053</td>\n",
              "      <td>17.699231</td>\n",
              "      <td>260.225396</td>\n",
              "      <td>0.828677</td>\n",
              "      <td>-1</td>\n",
              "      <td>4054.0</td>\n",
              "      <td>246.749827</td>\n",
              "      <td>0.948216</td>\n",
              "      <td>1.138875</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img012-00472</td>\n",
              "      <td>B</td>\n",
              "      <td>4358.5</td>\n",
              "      <td>89</td>\n",
              "      <td>78</td>\n",
              "      <td>6942</td>\n",
              "      <td>25.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.141026</td>\n",
              "      <td>22.225269</td>\n",
              "      <td>311.237588</td>\n",
              "      <td>0.627845</td>\n",
              "      <td>-1</td>\n",
              "      <td>5053.5</td>\n",
              "      <td>282.962375</td>\n",
              "      <td>0.909152</td>\n",
              "      <td>1.373701</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00554</td>\n",
              "      <td>B</td>\n",
              "      <td>3612.0</td>\n",
              "      <td>93</td>\n",
              "      <td>64</td>\n",
              "      <td>5952</td>\n",
              "      <td>32.0</td>\n",
              "      <td>17.5</td>\n",
              "      <td>1.453125</td>\n",
              "      <td>32.847826</td>\n",
              "      <td>344.450791</td>\n",
              "      <td>0.606855</td>\n",
              "      <td>-1</td>\n",
              "      <td>5469.0</td>\n",
              "      <td>289.761270</td>\n",
              "      <td>0.841227</td>\n",
              "      <td>1.088316</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  pred_y_rf  pred_y_kmeans_hog  pred_y_rf_hog\n",
              "0  img012-00236        B    4358.5  ...          B                  2              B\n",
              "1  img012-00584        B    3284.0  ...          B                  2              B\n",
              "2  img012-00406        B    3826.0  ...          B                  2              B\n",
              "3  img012-00472        B    4358.5  ...          B                  2              B\n",
              "4  img012-00554        B    3612.0  ...          B                  2              B\n",
              "\n",
              "[5 rows x 21 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 77
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "3HrsGruh9JO1",
        "colab_type": "code",
        "outputId": "2c3306d7-2a0b-4bcb-e0bc-a8c1b4d88900",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo RF_hog\n",
        "\n",
        "pd.crosstab(final_treino[\"ANOTACAO\"],final_treino[\"pred_y_rf_hog\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_rf_hog</th>\n",
              "      <th>A</th>\n",
              "      <th>B</th>\n",
              "      <th>C</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>A</th>\n",
              "      <td>100.0</td>\n",
              "      <td>0.000000</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>B</th>\n",
              "      <td>0.0</td>\n",
              "      <td>100.000000</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>C</th>\n",
              "      <td>0.0</td>\n",
              "      <td>0.125156</td>\n",
              "      <td>99.874844</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_rf_hog      A           B          C\n",
              "ANOTACAO                                   \n",
              "A              100.0    0.000000   0.000000\n",
              "B                0.0  100.000000   0.000000\n",
              "C                0.0    0.125156  99.874844"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 78
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "dGTIfuwT9SO8",
        "colab_type": "text"
      },
      "source": [
        "Como podemos notar, o modelo de HOG + Random Forest consegue 100% de assertividade no treino. Vamos fazer agora todas as aplicações na base de teste."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "7aQ9zLg-96hd",
        "colab_type": "text"
      },
      "source": [
        "**BASE DE TESTE**"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "H_OFOv4y9-Oj",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Para tirar o zip dos arquivos de teste\n",
        "!unzip teste_set"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "lzX-QAOi-kzd",
        "colab_type": "code",
        "outputId": "5de37bdf-956c-4de6-93ed-f04c4abd7672",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 35
        }
      },
      "source": [
        "# Inserindo todas as imagens na base x\n",
        "# Inserindo também as informações na base final para envio\n",
        "fil = glob.glob (\"/content/*.png\")\n",
        "for myFile in fil:\n",
        "    image = cv2.imread (myFile, 0)\n",
        "    x_teste.append (image) # Agregando a imagem no x\n",
        "    final_teste.loc[myFile, 'IMAGEM'] = myFile[9:-4] # Agregando o nome da imagem no arquivo final\n",
        "    final_teste.loc[myFile, 'ANOTACAO'] = myFile[9:15] # Agregando o nome da imagem (6 primeiro digitos)\n",
        "\n",
        "print('x_teste shape:', np.array(x_teste).shape) # Analisando o tamando da base"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "x_teste shape: (615, 128, 128)\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "WuZuRfMx-kve",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Resetando o index da base final\n",
        "final_teste = final_teste.reset_index(drop=True)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "BdCljCsTBKbo",
        "colab_type": "text"
      },
      "source": [
        "**INICIANDO OS TRATAMENTOS NA BASE DE TESTE**"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "xg8rsxIbBWoC",
        "colab": {}
      },
      "source": [
        "# Criando uma base para alocar as imagens alteradas\n",
        "x_teste_not = []"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "SN464k4fBWoV",
        "colab": {}
      },
      "source": [
        "# Alterando as cores da imagem e do fundo\n",
        "for image in x_teste:\n",
        "  img = cv2.bitwise_not(image)\n",
        "  x_teste_not.append(img)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "e7932f83-1df4-4063-bce9-b38610e11b43",
        "id": "9pVl1eFpBWom",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Teste de verificação\n",
        "plt.imshow(x_teste_not[150])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "<matplotlib.image.AxesImage at 0x7f4a4a762ef0>"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 84
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "iVBORw0KGgoAAAANSUhEUgAAATgAAAEyCAYAAABu5MwMAAAABHNCSVQICAgIfAhkiAAAAAlwSFlz\nAAALEgAACxIB0t1+/AAAADh0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uMy4xLjIsIGh0\ndHA6Ly9tYXRwbG90bGliLm9yZy8li6FKAAAY+klEQVR4nO3de5AV5ZnH8e8jIAjEIF4AAWU0QyLR\nGC3CRXRDBVPiZVW2jAurySSxZM26iRDLW/xDk0plN6VRWSuAJBhdY42yxAthjUaJZL2BgooXECEq\nd2SsgIooAXn2j+4eToYZBs5l3j7v+X2qqOnT3TP9VMM8PP3e2twdEZEYHRA6ABGRSlGCE5FoKcGJ\nSLSU4EQkWkpwIhItJTgRiVbFEpyZjTWz5Wa20syurdR1RETaYpUYB2dmnYA3ga8Da4EXgAnuvrTs\nFxMRaUOlKrhhwEp3f8vd/wbcB5xXoWuJiLSqc4V+bn9gTcHntcDwtk4+0Lp6N3pUKBQRidmHbH7P\n3Q9v7VilEly7zGwiMBGgG90ZbmNChSIiVewJn72qrWOVekRdBwws+Dwg3dfM3We4+1B3H9qFrhUK\nQ0RqWaUS3AtAvZnVmdmBwHhgToWuJSLSqoo8orr7TjP7d+AxoBNwp7u/XolriYi0pWJtcO7+CPBI\npX6+iEh7NJNBRKKlBCci0VKCE5FoKcGJSLSU4EQkWkpwIhItJTgRiZYSnIhESwlORKKlBCci0VKC\nE5FoKcGJSLSU4EQkWkpwIhItJTgRiZYSnIhESwlORKKlBCci0VKCE5FoKcGJSLSU4EQkWkpwIhIt\nJTgRiZYSnIhESwlORKKlBCci0VKCE5FoKcGJSLQ6hw5Aasv6q04B4NXJUwNHkqi/53sAHHPNc4Ej\nkUpQBSci0VIFJ2Wx/ayvAHDj7TMBGH3QrjbOfLmDIto3K745Ldn4ZtvnbN31CQCjbv4hAH1ve7bS\nYUmZqIITkWiZu4eOgYOttw+3MaHDkHZsbhgJQONPbgLg2C49Q4YTnCq7fHjCZy9296GtHVMFJyLR\nKrqCM7OBwH8DfQAHZrj7FDPrDdwPDALeAS509817+1mq4PJp40PHAbBkWGPgSKrPxe+MBqDplC1h\nA6kBlargdgJXuvsQYARwuZkNAa4F5rl7PTAv/Swi0uGK7kV19w3AhnT7QzNbBvQHzgNGp6fdDcwH\nrikpSukQe1Zs+erxrCa/HTQ/2ViffFFFF0ZZ2uDMbBBwErAQ6JMmP4CNJI+wIiIdruQEZ2Y9gd8B\nk9z9g8JjnjTwtdrIZ2YTzWyRmS3awfZSwxAR2UNJA33NrAtJcrvX3R9Id79rZv3cfYOZ9QM2tfa9\n7j4DmAFJJ0Mpccj+WzFlRPP2W9+Ynm7pkbRSWj6yDv5zAwB1E5aECahGFF3BmZkBM4Fl7n5LwaE5\nQEO63QA8XHx4IiLFK2WYyKnAU8CrQDYv50ck7XCzgKOAVSTDRP66t5+lYSKVt+9TqaQjabBw6fY2\nTKSUXtSnAWvjsLKViASnyfaRy9radrezSZ70PKAbAEuuTpaPGjy8ofmY2udKp6laIhItVXCRygbt\nvjVMlVs1efOrdzdvT1/aH4DGq88GoNvc54PEVM1UwYlItFTBReaohT0AeGygJshXu8t6rQPgn6bd\nCsC47lcC0HPWgmAxVRtVcCISLVVwVU7j2+J3RKekKn/wF78AYByq5PaVKjgRiZYquCqUVW2gyq2W\nqJLbf6rgRCRaquCqUP0NS5u3VbnVnqyS++aNvwegcZvGybVFFZyIREsJTkSipUfUKpIN4v3VwGcC\nR1JZmz79CIBxV5beiL7+qlMAeHXy1NIDy5lsIPDTP3oTgKa5IaPJJ1VwIhItvdm+CsS05FE5q7Ny\n2NwwEoDGn9zUvO/YLj1DhVOS+nu+B8Ax1zwXOJKOpTfbi0hNUgWXYzG0H1Xzy1X2fE9svrWsjiF8\nhdwRVMGJSE1SL2oOZVOxbr+s+trcWr7BvY7qq9wyfc9fBsAZfBnIf0WXDQCum/RG876mWaGiyQdV\ncCISLVVwObR54lYg/9OwsjYfyE+vaCVlFd2whqS3Mut5zVuv6/SjHm3eHjWptl9HqApORKKlXtQc\nqZZe05btbLUuzzNMauHvSr2oIlKT1AaXIyMuyHePYzWPaauk1cOTtshLF45q3peXai5rj6vVtjhV\ncCISLVVwOZC1vT02MJ9tb6rc9k1WycHuai50JdfzgG4AfFwQWy1RBSci0VKCE5Fo6RE1B/LauaBH\n0+Kt+PEQAObfnixdlPdB27FSBSci0VIFF1BeOxeywaGq3IrX9ZEXAPj+Fy8Dwg/efvG0O4DaGy5S\ncgVnZp3M7CUzm5t+rjOzhWa20szuN7MDSw9TRGT/laOCuwJYBhycfv45cKu732dm04FLgGlluE50\n8tb2Nn1LfwDW/GwwAN3QezZLdeRNSaV06QVhh43U6nCRkio4MxsAnA38Ov1swNeA2ekpdwPnl3IN\nEZFilVrB3QZcDXwm/XwosMXdd6af1wL9S7xGVLKXnABM7Zu96CTscjtbd30CwLQZ5wHQd25ttM90\npAWzT0w2JudjCletKLqCM7NzgE3uvrjI759oZovMbNEOthcbhohIm0qp4EYB55rZWUA3kja4KUAv\nM+ucVnEDgHWtfbO7zwBmQLJcUglxVJXt43YvW5OXhRJPfupfAairkZ61EPLSFjfs6FUANAW5escr\nuoJz9+vcfYC7DwLGA39y94uAJ4EL0tMagIdLjlJEpAiVGAd3DXCfmf0UeAmYWYFrSIkKlxs//MGD\nAkZSW+YtOCHZCFTBndprBQCN55wNQLe5cfeUlyXBuft8YH66/RYwrBw/V0SkFJrJ0EGalyMflo9Z\nC6c+/W/N23URvygmb3qsDTs78uKD/wLAtM8lv/p9QwbTATQXVUSipQqugxwwanPoEIDdbW9qdwvj\n0Nd3ADD/46S20CojlaUKTkSipQQnItHSI2qNyToX1LEgtUAVnIhESxVchWWT6x84KezE+mxC/UHp\nW9gljGwhzP945EvJ10Bx9KU2puWpghORaKmCq7D3Tk7WEQg9sf6y1WOB2lmqWgRUwYlIxFTB1YiF\nTx0HwDE8FzgSkY6jCk5EoqUKrsI+c/T7Qa+fTc06YnHNrCkq0kwVnIhESxVcheRl/NsDHyavAOy8\nTZO6pfaoghORaKmCq5CPjzAg/Pi3W5acDkBd5EtTi7RGFZyIREsVXOR8dffQIYgEowpORKKlBCci\n0dIjaoV8NCDssAwN8BVRBSciEVMFVyGhp2hpgK+IKjgRiZgquAoZ1m910Os/vaUegG4a4Cs1TBWc\niERLFVyZbT/rKwBcdNjMoHE8v+poAOrYEjQOkZBUwYlItFTBldm2w5NbOrDzB+mesJPtRWqZKjgR\niZYquEhpkr2IKjgRiVhJCc7MepnZbDN7w8yWmdlIM+ttZo+b2Yr06yHlClbat3XXJ2zd9QndNxjd\nN1jocESCKrWCmwI86u5fAE4ElgHXAvPcvR6Yl34WEelwRSc4M/ss8A/ATAB3/5u7bwHOA+5OT7sb\nOL/UIEVEilFKBVcHNAG/MbOXzOzXZtYD6OPuG9JzNgJ9WvtmM5toZovMbNEOtpcQhohI60pJcJ2B\nk4Fp7n4S8BEtHkfd3YFWFyRz9xnuPtTdh3ahawlhiIi0rpRhImuBte6+MP08myTBvWtm/dx9g5n1\nAzaVGmQ1Cf02rW3+KQA912uZJJGiKzh33wisMbPPp7vGAEuBOUBDuq8BeLikCEVEilTqQN/vA/ea\n2YHAW8B3SJLmLDO7BFgFXFjiNUREilJSgnP3l4GhrRwaU8rPFREpB81kEJFoKcGJSLSU4EQkWkpw\nIhItJTgRiZYSnIhESwlORKKlBCci0VKCE5FoKcGJSLT00pnIdLdOAGw9Mvm/Sy8tlFqmCk5EoqUE\nJyLR0iNqmR20KVnA+C87tgLhFr4UEVVwIhIxJTgRiZYSnIhES21wkel5QDcAtvVr9WVmIjVFFZyI\nREsVXJl1b9oJwJqdBwNwbJcwr++zo7YFua5InqiCE5FoKcGJSLSU4EQkWmqDK7Ouj7wAwL03jARg\n9MBngsQx7OhVADQFubpIPqiCE5FoqYKrkOc3HJVsBKrgph/1KACjJv0QgL63PRskDpGQVMGJSLRU\nwVXIh6s+m2wMC3N9zWgQUQUnIhFTghORaOkRtUJ6rNX/HSKh6bdQRKKlCq5C8rJ0uSbdSy0rqYIz\ns8lm9rqZvWZmjWbWzczqzGyhma00s/vN7MByBSsisj+KTnBm1h/4ATDU3Y8HOgHjgZ8Dt7r754DN\nwCXlCLTadG/aSfemnazZeXDz0kkhvHjaHbx42h1snHQKGyedEiwOkRBKbYPrDBxkZp2B7sAG4GvA\n7PT43cD5JV5DRKQoRbfBufs6M7sZWA18DPwRWAxscfed6Wlrgf4lR1mF8jLpPhvw+/Hwj4JcXySk\nUh5RDwHOA+qAI4EewNj9+P6JZrbIzBbtYHuxYYiItKmUXtTTgbfdvQnAzB4ARgG9zKxzWsUNANa1\n9s3uPgOYAXCw9Y52PtG8BSckG4EqOMmX7Wd9BYAbb58JwOiDOnZJ+627PgFg1M21sQhDKW1wq4ER\nZtbdzAwYAywFngQuSM9pAB4uLUQRkeKU0ga30MxmAy8CO4GXSCqy/wXuM7OfpvtmliPQapWXGQ0v\nnnYHoOWTpLaUNNDX3W8Abmix+y2CraEhIrKbZjJU2KGv7wBg/sdJJdfRbS6ZrDf1hAuXAtB0W5Aw\nat62w5NfuYGdP0j3hJnhUivy8fwkIlIBquAqLBsPd8XEfwZgybDGkOFIYO+dnAwYCDU3eZt/CkDP\n9WGeJDqaKjgRiZYSnIhES4+oHWTXM4ckG4H7l387aD4AgxsbmvfVTVgSKBqRylIFJyLRUgXXQVoO\nF4FwQ0YAnj51avP2uAuvBKDnrAWhwqkZnzn6/aDXf+DDwQB03qZOBhGRqqYKroO0HC4CYYeMHNGp\nR/N23aQ3AGiaFSqa+K2/Klls9NVhU9s5s7JuWXI6AHVznw8aR0dRBSci0VIF18Gae1MheI9qJutZ\nrf/59wA45prnAkYTpxEX5KOn2ld3Dx1Ch1IFJyLRUgXXwY68afcyRZdeMAqAX+VkMcxn/uVmAMYt\nVq9quWRtb48NDNv2li102X2DBY2jo6mCE5FoqYILaMHsE5ONyfmo4LKeVfWqli5bmvz2y6YHjiTx\n2w+OBaDXyp3tnBkXVXAiEi1VcAHlZTHMllrOV9Vc1f23eeJWID9/p7U2/i2jCk5EoqUKLqBsdsN3\nz5gIwFvfyEd7TebNr94NqJLbV1mPKYSfsZDJek8PWtijnTPjpApORKKlBCci0dIjag7UX5EMqL10\nRL4G/mb0qLp3zRPpJ+fjsbRQrQ4PyaiCE5FoqYLLkbwN/G0pq+QufnY0AE2nbAkYTXh5rtwyv/j9\nuQAcM7c2F1BQBSci0VIFlyPZRPwTR00A8vsO1Wwg8KY1HzXvG3dl7UzQ3/jQcUB+hoK0NH1L/+bt\nI5/6NGAk4amCE5FoqYLLoUNmJG89n39CvqZwtVS47PkztyWDlC+eNBqIq31uc8NIABp/chMAx3Z5\nOWQ47cra3aB2294yquBEJFrm7qFj4GDr7cNtTOgwcqcaeunaM/jP1Td2Llvq6MbbZwL5raBbuvid\n0UBc1fO+eMJnL3b3oa0dUwUnItFSG1yOVUuv6t5kY+dYn3zZ9GnS85qXXtcVU0YALRc6yHcbW0vZ\nhPpXZw0BoC/P7u30mqIKTkSi1W4bnJndCZwDbHL349N9vYH7gUHAO8CF7r7ZzAyYApwFbAO+7e4v\ntheE2uD2zVHpkjd5m6tabtk4rsarzwagWwmLNLZeocWl/p7aft1jqW1wdwFjW+y7Fpjn7vXAvPQz\nwJlAffpnIjCtmIBFRMphn3pRzWwQMLegglsOjHb3DWbWD5jv7p83szvS7caW5+3t56uC2zcte/eg\nenr4pPxqtde0pUr0ovYpSFobgT7pdn9gTcF5a9N9ezCziWa2yMwW7WB7kWGIiLSt5E4GT0rA/R5M\n5+4z3H2ouw/tQtdSwxAR2UOxw0TeNbN+BY+om9L964CBBecNSPdJGWTvcLiRS5r3VdtgVCld1gmz\n5meDAehGbb0pa38UW8HNARrS7Qbg4YL937LECOD99trfREQqpd0KzswagdHAYWa2FrgB+E9glpld\nAqwCLkxPf4RkiMhKkmEi36lAzDUvq+RgdzWnSi5+2SDpe278RwB6zo1/aapStZvg3H1CG4f26PZM\n2+MuLzUoEZFy0FStKteyXU6VXHzyNr2tmmiqlohES8slRapWpnXFrJxT1mKm5ZJEpCapDS5Sq4cn\n7TbHTLkMiHuyeUyy6VewewqWxrkVTxWciERLFVzk6q9IetyGvZgsqbP7xSk9g8Uke6r1JY8qRRWc\niERLvag1KlsIEtQ+F4J6SMtHvagiUpNUwUkzjZ2rnOzFMKNu/iEAfW/Ti2HKRRWciNQkJTgRiZaG\niUizbHDwGXwZgI0PHQdU5/tYQ2vrkVTvLO1YquBEJFqq4KRNfc9fBuyu6NZfdQoAr06eGiymPNrb\n261UsYWlCk5EoqVhIlIWmxtGAnFPBRv85+Q1JHUTlgSORAppmIiI1CRVcNKhsilieZkeprfDVz9V\ncCJSk1TBiUhVUwUnIjVJCU5EoqUEJyLRUoITkWgpwYlItJTgRCRaSnAiEi0lOBGJlhKciERLCU5E\noqUEJyLRajfBmdmdZrbJzF4r2HeTmb1hZq+Y2YNm1qvg2HVmttLMlpvZGZUKXESkPftSwd0FjG2x\n73HgeHf/EvAmcB2AmQ0BxgNfTL9nqpl1Klu0IiL7od0E5+7/B/y1xb4/uvvO9OMCYEC6fR5wn7tv\nd/e3gZXAsDLGKyKyz8rRBvdd4A/pdn9gTcGxtek+EZEOV9JbtczsemAncG8R3zsRmAjQje6lhCEi\n0qqiE5yZfRs4Bxjju1fNXAcMLDhtQLpvD+4+A5gByYKXxcYhItKWoh5RzWwscDVwrrtvKzg0Bxhv\nZl3NrA6oB54vPUwRkf3XbgVnZo3AaOAwM1sL3EDSa9oVeNzMABa4+2Xu/rqZzQKWkjy6Xu7un1Yq\neBGRvdE7GUSkqumdDCJSk5TgRCRaSnAiEi0lOBGJlhKciERLCU5EoqUEJyLRUoITkWgpwYlItJTg\nRCRaSnAiEq1czEU1sybgI+C90LHso8OojlirJU6onlirJU6onlhLjfNodz+8tQO5SHAAZraorQmz\neVMtsVZLnFA9sVZLnFA9sVYyTj2iiki0lOBEJFp5SnAzQgewH6ol1mqJE6on1mqJE6on1orFmZs2\nOBGRcstTBSciUla5SHBmNtbMlpvZSjO7NnQ8GTMbaGZPmtlSM3vdzK5I9/c2s8fNbEX69ZDQsQKY\nWScze8nM5qaf68xsYXpf7zezA0PHCGBmvcxstpm9YWbLzGxkju/p5PTv/jUzazSzbnm4r2Z2p5lt\nMrPXCva1eg8t8V9pvK+Y2ck5iPWm9O//FTN70Mx6FRy7Lo11uZmdUcq1gyc4M+sE/BI4ExgCTDCz\nIWGjarYTuNLdhwAjgMvT2K4F5rl7PTAv/ZwHVwDLCj7/HLjV3T8HbAYuCRLVnqYAj7r7F4ATSWLO\n3T01s/7AD4Ch7n480AkYTz7u613A2Bb72rqHZ5K84a6e5F3E0zooxsxd7Bnr48Dx7v4l4E2SF1mR\n/n6NB76Yfs/UNEcUx92D/gFGAo8VfL4OuC50XG3E+jDwdWA50C/d1w9YnoPYBpD8o/4aMBcwksGT\nnVu7zwHj/CzwNmn7b8H+PN7T/sAaoDfJG+jmAmfk5b4Cg4DX2ruHwB3AhNbOCxVri2PjgHvT7b/7\n/QceA0YWe93gFRy7/xFl1qb7csXMBgEnAQuBPu6+IT20EegTKKxCt5G8q3ZX+vlQYIu770w/5+W+\n1gFNwG/Sx+lfm1kPcnhP3X0dcDOwGtgAvA8sJp/3Fdq+h3n/Hfsu8Id0u6yx5iHB5Z6Z9QR+B0xy\n9w8Kj3ny30zQrmgzOwfY5O6LQ8axjzoDJwPT3P0kkil6f/c4mod7CpC2YZ1HkpSPBHqw56NWLuXl\nHrbHzK4naQq6txI/Pw8Jbh0wsODzgHRfLphZF5Lkdq+7P5DuftfM+qXH+wGbQsWXGgWca2bvAPeR\nPKZOAXqZWfZy77zc17XAWndfmH6eTZLw8nZPAU4H3nb3JnffATxAcq/zeF+h7XuYy98xM/s2cA5w\nUZqQocyx5iHBvQDUpz1TB5I0MM4JHBOQ9D4BM4Fl7n5LwaE5QEO63UDSNheMu1/n7gPcfRDJ/fuT\nu18EPAlckJ4WPE4Ad98IrDGzz6e7xgBLydk9Ta0GRphZ9/TfQhZr7u5rqq17OAf4VtqbOgJ4v+BR\nNggzG0vSpHKuu28rODQHGG9mXc2sjqRj5PmiLxSicbSVRsazSHpS/gJcHzqegrhOJSnzXwFeTv+c\nRdK+NQ9YATwB9A4da0HMo4G56fYx6T+OlcD/AF1Dx5fG9WVgUXpfHwIOyes9BX4MvAG8BtwDdM3D\nfQUaSdoFd5BUxZe0dQ9JOpx+mf5+vUrSKxw61pUkbW3Z79X0gvOvT2NdDpxZyrU1k0FEopWHR1QR\nkYpQghORaCnBiUi0lOBEJFpKcCISLSU4EYmWEpyIREsJTkSi9f93HpFnHqBunwAAAABJRU5ErkJg\ngg==\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "wPkZiDOnBwPC"
      },
      "source": [
        "Aplicando o esqueleto"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "Ekpn99vVBwPc",
        "colab": {}
      },
      "source": [
        "#Base com esqueleto\n",
        "\n",
        "x_teste_esq = []\n",
        "count = 0\n",
        "\n",
        "for image in x_teste_not:\n",
        "  img_skel = image\n",
        "  _, img_skel = cv2.threshold(img_skel, 127, 255, cv2.THRESH_OTSU)\n",
        "  skeleton_new = zhangSuen(img_skel)\n",
        "  x_teste_esq.append(skeleton_new) \n",
        "  print(count)\n",
        "  count += 1"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "9d5ddc6c-456c-4f64-feb6-0262778dd971",
        "id": "_kw0EM2GBwPw",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Testando uma imagem com esqueleto\n",
        "plt.imshow(x_teste_esq[150])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "<matplotlib.image.AxesImage at 0x7f4a4a753780>"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 86
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "pwzTRjvPF4RN"
      },
      "source": [
        "**INSERINDO OS DESCRITORES NA BASE DE TESTE**"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "flVTQxm3F4Re"
      },
      "source": [
        "Aplicando a área das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "FdeKJ0LsF4Ro",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as áreas\n",
        "base_area = []\n",
        "for img in x_teste_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  area = cv2.contourArea(biggest)\n",
        "  base_area.append(area)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "GSxs1WW3F4SL",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_area)).rename(columns={0:'AREA_OBJ'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "6mjbzByLGT_N"
      },
      "source": [
        "Aplicando Altura das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "chGtJTdsGT_i",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as alturas\n",
        "base_altura = []\n",
        "for alt in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(alt)\n",
        "  altura = h\n",
        "  base_altura.append(altura)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "UKeX5Gs3GUAJ",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de altura com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_altura)).rename(columns={0:'ALTURA'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "6lNIufgKGUAp"
      },
      "source": [
        "Aplicando a largura das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "1QN9RAjRGUAt",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as larguras\n",
        "base_largura = []\n",
        "for lar in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(lar)\n",
        "  largura = w\n",
        "  base_largura.append(largura)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "SvLo42Q8GUA5",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de largura com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_largura)).rename(columns={0:'LARGURA'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "i4phgxzJGUBG"
      },
      "source": [
        "Aplicando a área do contorno"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "waKSvLN8GUBI",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as areas\n",
        "base_area2 = []\n",
        "for area in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(area)\n",
        "  objeto = w * h\n",
        "  base_area2.append(objeto)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "n6Jc78PFGUBO",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_area2)).rename(columns={0:'AREA_CONTORNO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "jhro6YqmG-u0"
      },
      "source": [
        "Aplicando balanceamento horizontal"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "bAtAl1lQG-vO",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o balanceamento horizontal\n",
        "balan_hor = []\n",
        "for hor in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(hor)\n",
        "  horizontal = (64-(w/2))\n",
        "  balan_hor.append(horizontal)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "MDZVizsqG-vk",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de balanceamento horizontal com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(balan_hor)).rename(columns={0:'BALAN_HOR'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "bVk-gvEdG-v1"
      },
      "source": [
        "Aplicando balanceamento vertical"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "wqFV4Y4NG-v3",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o balanceamento vertical\n",
        "balan_ver = []\n",
        "for ver in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(ver)\n",
        "  vertical = (64-(h/2))\n",
        "  balan_ver.append(vertical)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "gLxbjxz1G-v-",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de balanceamento vertical com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(balan_ver)).rename(columns={0:'BALAN_VER'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "rehWa2p-G-wH"
      },
      "source": [
        "Aplicando a razão das imagens"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "9nX6RTIGG-wJ",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as razões\n",
        "base_razao = []\n",
        "for raz in x_teste_esq:\n",
        "  x, y, w, h = cv2.boundingRect(raz)\n",
        "  razao = h/w\n",
        "  base_razao.append(razao)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "gxfF880DG-wO",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de razão com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_razao)).rename(columns={0:'RAZAO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "j27JKReSG-wa"
      },
      "source": [
        "Aplicando a compacidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "j9RAoBnBG-wc",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir a compacidade\n",
        "base_compacidade = []\n",
        "\n",
        "for img in x_teste_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  area = cv2.contourArea(biggest)\n",
        "  perimetro = cv2.arcLength(biggest, True)\n",
        "  compacidade = ((perimetro**2)/area) if area != 0 else 0\n",
        "  base_compacidade.append(compacidade)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "xhGTRWwPG-wh",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de compacidade com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_compacidade)).rename(columns={0:'COMPACIDADE'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "nEmdaHgcG-wq"
      },
      "source": [
        "Aplicando o perímetro do contorno"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "DPMPE8MQG-wq",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o perímetro do contorno\n",
        "base_perimetro = []\n",
        "\n",
        "for img in x_teste_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  perimetro = cv2.arcLength(biggest, True)\n",
        "  base_perimetro.append(perimetro)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "STC02uFXG-wu",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de perímetro com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_perimetro)).rename(columns={0:'PERIM_CONTORNO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "TnyL5rFXH2vO"
      },
      "source": [
        "Aplicando a retangularidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "ZH8xEPP1H2vl",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de área, largura e altura calculados na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_teste['RETANG.'] = final_teste['AREA_OBJ'] / (final_teste['ALTURA'] * final_teste['LARGURA'])"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "Q-MSnQ3uH2wI"
      },
      "source": [
        "Aplicando o número de Euler"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "XS6uYkMpH2wM",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir o número de Euler\n",
        "base_euler = []\n",
        "\n",
        "for img in x_teste_esq:\n",
        "  contornos,_ = cv2.findContours(img,cv2.RETR_CCOMP,cv2.CHAIN_APPROX_SIMPLE)\n",
        "  euler = 2 - len(contornos) # Tiramos 2 do contorno pois 1 representa a letra em si e o outro 1 é da fórmula.\n",
        "  base_euler.append(euler)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "kTbstp0dH2wX",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de Euler com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_euler)).rename(columns={0:'EULER'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "C8MvXs2iH2wp"
      },
      "source": [
        "Aplicando a área e perimetro do fecho convexo"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "ml6WSk_oH2wr",
        "colab": {}
      },
      "source": [
        "# Criando a base para inserir as áreas\n",
        "base_area3 = []\n",
        "base_perimetro2 = []\n",
        "\n",
        "for img in x_teste_esq:\n",
        "  _, binary_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)\n",
        "  cnt, hierarchy = cv2.findContours(binary_otsu, 1, 2)\n",
        "  biggest = max(cnt, key = cv2.contourArea)\n",
        "  hull = cv2.convexHull(biggest)\n",
        "  area = cv2.contourArea(hull)\n",
        "  base_area3.append(area)\n",
        "  perimetro = cv2.arcLength(hull, True)\n",
        "  base_perimetro2.append(perimetro)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "O0Gru5hnH2wx",
        "colab": {}
      },
      "source": [
        "# Cruzando as informações de área e perímetro do fecho com as já existentes\n",
        "final_teste = final_teste.join(pd.DataFrame(base_area3)).rename(columns={0:'AREA_FECHO'})\n",
        "final_teste = final_teste.join(pd.DataFrame(base_perimetro2)).rename(columns={0:'PERIM_FECHO'})"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "t6_RgyZLH2w-"
      },
      "source": [
        "Aplicando a convexidade"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "s43Our1kH2w_",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de perímetro do fecho e do contorno na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_teste['CONVEXIDADE'] = final_teste['PERIM_FECHO'] / final_teste['PERIM_CONTORNO']"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "yI-P8YQVH2xM"
      },
      "source": [
        "Aplicando a solidez"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "A7dMS_evH2xP",
        "colab": {}
      },
      "source": [
        "# Como já temos os valores de área do fecho e do contorno na tabela, vamos fazer o cálculo diretamente\n",
        "\n",
        "final_teste['SOLIDEZ'] = final_teste['AREA_CONTORNO'] / final_teste['AREA_FECHO']"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "30fb4012-f078-48a8-c36b-801ea815ffcd",
        "id": "TUAYkPpGH2xT",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_teste.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img013-00999</td>\n",
              "      <td>img013</td>\n",
              "      <td>262.5</td>\n",
              "      <td>68</td>\n",
              "      <td>95</td>\n",
              "      <td>6460</td>\n",
              "      <td>16.5</td>\n",
              "      <td>30.0</td>\n",
              "      <td>0.715789</td>\n",
              "      <td>792.535990</td>\n",
              "      <td>456.114785</td>\n",
              "      <td>0.040635</td>\n",
              "      <td>-1</td>\n",
              "      <td>2508.0</td>\n",
              "      <td>240.611816</td>\n",
              "      <td>0.527525</td>\n",
              "      <td>2.575758</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img013-00985</td>\n",
              "      <td>img013</td>\n",
              "      <td>19.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>9626.458311</td>\n",
              "      <td>433.261973</td>\n",
              "      <td>0.003121</td>\n",
              "      <td>1</td>\n",
              "      <td>5304.0</td>\n",
              "      <td>265.673660</td>\n",
              "      <td>0.613194</td>\n",
              "      <td>1.177979</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img013-00983</td>\n",
              "      <td>img013</td>\n",
              "      <td>25.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>8000.878151</td>\n",
              "      <td>451.688380</td>\n",
              "      <td>0.004081</td>\n",
              "      <td>1</td>\n",
              "      <td>5037.0</td>\n",
              "      <td>256.844223</td>\n",
              "      <td>0.568631</td>\n",
              "      <td>1.240421</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img011-00856</td>\n",
              "      <td>img011</td>\n",
              "      <td>978.0</td>\n",
              "      <td>64</td>\n",
              "      <td>86</td>\n",
              "      <td>5504</td>\n",
              "      <td>21.0</td>\n",
              "      <td>32.0</td>\n",
              "      <td>0.744186</td>\n",
              "      <td>144.967169</td>\n",
              "      <td>376.534051</td>\n",
              "      <td>0.177689</td>\n",
              "      <td>0</td>\n",
              "      <td>3297.0</td>\n",
              "      <td>243.667625</td>\n",
              "      <td>0.647133</td>\n",
              "      <td>1.669396</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00847</td>\n",
              "      <td>img012</td>\n",
              "      <td>4432.5</td>\n",
              "      <td>89</td>\n",
              "      <td>82</td>\n",
              "      <td>7298</td>\n",
              "      <td>23.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.085366</td>\n",
              "      <td>21.757949</td>\n",
              "      <td>310.551297</td>\n",
              "      <td>0.607358</td>\n",
              "      <td>-1</td>\n",
              "      <td>4837.0</td>\n",
              "      <td>280.695008</td>\n",
              "      <td>0.903860</td>\n",
              "      <td>1.508786</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  PERIM_FECHO  CONVEXIDADE   SOLIDEZ\n",
              "0  img013-00999   img013     262.5  ...   240.611816     0.527525  2.575758\n",
              "1  img013-00985   img013      19.5  ...   265.673660     0.613194  1.177979\n",
              "2  img013-00983   img013      25.5  ...   256.844223     0.568631  1.240421\n",
              "3  img011-00856   img011     978.0  ...   243.667625     0.647133  1.669396\n",
              "4  img012-00847   img012    4432.5  ...   280.695008     0.903860  1.508786\n",
              "\n",
              "[5 rows x 17 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 125
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "_mNrl5qWIwFL"
      },
      "source": [
        "Nrmalizando a base"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "7da1280e-79a3-4e52-c536-5ac111e966b5",
        "id": "GMuITB4eIwF_",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 206
        }
      },
      "source": [
        "# Escalando a base de teste\n",
        "\n",
        "final_teste_scaled = pd.DataFrame(scaler.transform(final_teste.iloc[:,3:17]))\n",
        "final_teste_scaled.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "      <th>3</th>\n",
              "      <th>4</th>\n",
              "      <th>5</th>\n",
              "      <th>6</th>\n",
              "      <th>7</th>\n",
              "      <th>8</th>\n",
              "      <th>9</th>\n",
              "      <th>10</th>\n",
              "      <th>11</th>\n",
              "      <th>12</th>\n",
              "      <th>13</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>-2.476937</td>\n",
              "      <td>1.672933</td>\n",
              "      <td>0.191017</td>\n",
              "      <td>-1.672933</td>\n",
              "      <td>2.476937</td>\n",
              "      <td>-1.462436</td>\n",
              "      <td>-0.263656</td>\n",
              "      <td>0.978087</td>\n",
              "      <td>-0.820133</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>-1.632038</td>\n",
              "      <td>-0.938632</td>\n",
              "      <td>-1.441277</td>\n",
              "      <td>0.058485</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>0.171420</td>\n",
              "      <td>-0.068661</td>\n",
              "      <td>0.021622</td>\n",
              "      <td>0.068661</td>\n",
              "      <td>-0.171420</td>\n",
              "      <td>-0.079860</td>\n",
              "      <td>0.628012</td>\n",
              "      <td>0.706947</td>\n",
              "      <td>-0.955126</td>\n",
              "      <td>0.866302</td>\n",
              "      <td>1.061982</td>\n",
              "      <td>0.119492</td>\n",
              "      <td>-0.835836</td>\n",
              "      <td>-0.047623</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>0.171420</td>\n",
              "      <td>-0.068661</td>\n",
              "      <td>0.021622</td>\n",
              "      <td>0.068661</td>\n",
              "      <td>-0.171420</td>\n",
              "      <td>-0.079860</td>\n",
              "      <td>0.463931</td>\n",
              "      <td>0.925569</td>\n",
              "      <td>-0.951670</td>\n",
              "      <td>0.866302</td>\n",
              "      <td>0.804720</td>\n",
              "      <td>-0.253291</td>\n",
              "      <td>-1.150768</td>\n",
              "      <td>-0.042883</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>-3.006609</td>\n",
              "      <td>1.019835</td>\n",
              "      <td>-0.572860</td>\n",
              "      <td>-1.019835</td>\n",
              "      <td>3.006609</td>\n",
              "      <td>-1.387461</td>\n",
              "      <td>-0.329020</td>\n",
              "      <td>0.033893</td>\n",
              "      <td>-0.326944</td>\n",
              "      <td>0.140498</td>\n",
              "      <td>-0.871816</td>\n",
              "      <td>-0.809614</td>\n",
              "      <td>-0.595983</td>\n",
              "      <td>-0.010319</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>0.303837</td>\n",
              "      <td>0.729570</td>\n",
              "      <td>0.860608</td>\n",
              "      <td>-0.729570</td>\n",
              "      <td>-0.303837</td>\n",
              "      <td>-0.486650</td>\n",
              "      <td>-0.341456</td>\n",
              "      <td>-0.748966</td>\n",
              "      <td>1.219218</td>\n",
              "      <td>-0.585307</td>\n",
              "      <td>0.612015</td>\n",
              "      <td>0.753701</td>\n",
              "      <td>1.218356</td>\n",
              "      <td>-0.022511</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         0         1         2   ...        11        12        13\n",
              "0 -2.476937  1.672933  0.191017  ... -0.938632 -1.441277  0.058485\n",
              "1  0.171420 -0.068661  0.021622  ...  0.119492 -0.835836 -0.047623\n",
              "2  0.171420 -0.068661  0.021622  ... -0.253291 -1.150768 -0.042883\n",
              "3 -3.006609  1.019835 -0.572860  ... -0.809614 -0.595983 -0.010319\n",
              "4  0.303837  0.729570  0.860608  ...  0.753701  1.218356 -0.022511\n",
              "\n",
              "[5 rows x 14 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 126
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "a-NIWJR7Jh2A",
        "colab": {}
      },
      "source": [
        "# Rodando o predict e já agregando na base final\n",
        "\n",
        "final_teste['pred_y_kmeans'] = kmeans.predict(final_teste_scaled)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "f319d80b-0fe6-42c9-ce02-f62f7e1df556",
        "id": "7AS9o1XaJh2Q",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_teste.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img013-00999</td>\n",
              "      <td>img013</td>\n",
              "      <td>262.5</td>\n",
              "      <td>68</td>\n",
              "      <td>95</td>\n",
              "      <td>6460</td>\n",
              "      <td>16.5</td>\n",
              "      <td>30.0</td>\n",
              "      <td>0.715789</td>\n",
              "      <td>792.535990</td>\n",
              "      <td>456.114785</td>\n",
              "      <td>0.040635</td>\n",
              "      <td>-1</td>\n",
              "      <td>2508.0</td>\n",
              "      <td>240.611816</td>\n",
              "      <td>0.527525</td>\n",
              "      <td>2.575758</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img013-00985</td>\n",
              "      <td>img013</td>\n",
              "      <td>19.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>9626.458311</td>\n",
              "      <td>433.261973</td>\n",
              "      <td>0.003121</td>\n",
              "      <td>1</td>\n",
              "      <td>5304.0</td>\n",
              "      <td>265.673660</td>\n",
              "      <td>0.613194</td>\n",
              "      <td>1.177979</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img013-00983</td>\n",
              "      <td>img013</td>\n",
              "      <td>25.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>8000.878151</td>\n",
              "      <td>451.688380</td>\n",
              "      <td>0.004081</td>\n",
              "      <td>1</td>\n",
              "      <td>5037.0</td>\n",
              "      <td>256.844223</td>\n",
              "      <td>0.568631</td>\n",
              "      <td>1.240421</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img011-00856</td>\n",
              "      <td>img011</td>\n",
              "      <td>978.0</td>\n",
              "      <td>64</td>\n",
              "      <td>86</td>\n",
              "      <td>5504</td>\n",
              "      <td>21.0</td>\n",
              "      <td>32.0</td>\n",
              "      <td>0.744186</td>\n",
              "      <td>144.967169</td>\n",
              "      <td>376.534051</td>\n",
              "      <td>0.177689</td>\n",
              "      <td>0</td>\n",
              "      <td>3297.0</td>\n",
              "      <td>243.667625</td>\n",
              "      <td>0.647133</td>\n",
              "      <td>1.669396</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00847</td>\n",
              "      <td>img012</td>\n",
              "      <td>4432.5</td>\n",
              "      <td>89</td>\n",
              "      <td>82</td>\n",
              "      <td>7298</td>\n",
              "      <td>23.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.085366</td>\n",
              "      <td>21.757949</td>\n",
              "      <td>310.551297</td>\n",
              "      <td>0.607358</td>\n",
              "      <td>-1</td>\n",
              "      <td>4837.0</td>\n",
              "      <td>280.695008</td>\n",
              "      <td>0.903860</td>\n",
              "      <td>1.508786</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  CONVEXIDADE   SOLIDEZ  pred_y_kmeans\n",
              "0  img013-00999   img013     262.5  ...     0.527525  2.575758              1\n",
              "1  img013-00985   img013      19.5  ...     0.613194  1.177979              0\n",
              "2  img013-00983   img013      25.5  ...     0.568631  1.240421              0\n",
              "3  img011-00856   img011     978.0  ...     0.647133  1.669396              1\n",
              "4  img012-00847   img012    4432.5  ...     0.903860  1.508786              0\n",
              "\n",
              "[5 rows x 18 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 128
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "WUxn44pCJh2b"
      },
      "source": [
        "Aplicando Random Forest no treino"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "juHJ8r4rJh2l",
        "colab": {}
      },
      "source": [
        "# Predict na base e já agregando a coluna ao final\n",
        "\n",
        "final_teste['pred_y_rf'] = model_rf.predict(final_teste_scaled)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "3056a374-465d-45bf-f5d2-797fce87abae",
        "id": "Nz7AgY2QJh2r",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_teste.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img013-00999</td>\n",
              "      <td>img013</td>\n",
              "      <td>262.5</td>\n",
              "      <td>68</td>\n",
              "      <td>95</td>\n",
              "      <td>6460</td>\n",
              "      <td>16.5</td>\n",
              "      <td>30.0</td>\n",
              "      <td>0.715789</td>\n",
              "      <td>792.535990</td>\n",
              "      <td>456.114785</td>\n",
              "      <td>0.040635</td>\n",
              "      <td>-1</td>\n",
              "      <td>2508.0</td>\n",
              "      <td>240.611816</td>\n",
              "      <td>0.527525</td>\n",
              "      <td>2.575758</td>\n",
              "      <td>1</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img013-00985</td>\n",
              "      <td>img013</td>\n",
              "      <td>19.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>9626.458311</td>\n",
              "      <td>433.261973</td>\n",
              "      <td>0.003121</td>\n",
              "      <td>1</td>\n",
              "      <td>5304.0</td>\n",
              "      <td>265.673660</td>\n",
              "      <td>0.613194</td>\n",
              "      <td>1.177979</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img013-00983</td>\n",
              "      <td>img013</td>\n",
              "      <td>25.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>8000.878151</td>\n",
              "      <td>451.688380</td>\n",
              "      <td>0.004081</td>\n",
              "      <td>1</td>\n",
              "      <td>5037.0</td>\n",
              "      <td>256.844223</td>\n",
              "      <td>0.568631</td>\n",
              "      <td>1.240421</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img011-00856</td>\n",
              "      <td>img011</td>\n",
              "      <td>978.0</td>\n",
              "      <td>64</td>\n",
              "      <td>86</td>\n",
              "      <td>5504</td>\n",
              "      <td>21.0</td>\n",
              "      <td>32.0</td>\n",
              "      <td>0.744186</td>\n",
              "      <td>144.967169</td>\n",
              "      <td>376.534051</td>\n",
              "      <td>0.177689</td>\n",
              "      <td>0</td>\n",
              "      <td>3297.0</td>\n",
              "      <td>243.667625</td>\n",
              "      <td>0.647133</td>\n",
              "      <td>1.669396</td>\n",
              "      <td>1</td>\n",
              "      <td>A</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00847</td>\n",
              "      <td>img012</td>\n",
              "      <td>4432.5</td>\n",
              "      <td>89</td>\n",
              "      <td>82</td>\n",
              "      <td>7298</td>\n",
              "      <td>23.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.085366</td>\n",
              "      <td>21.757949</td>\n",
              "      <td>310.551297</td>\n",
              "      <td>0.607358</td>\n",
              "      <td>-1</td>\n",
              "      <td>4837.0</td>\n",
              "      <td>280.695008</td>\n",
              "      <td>0.903860</td>\n",
              "      <td>1.508786</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...   SOLIDEZ  pred_y_kmeans  pred_y_rf\n",
              "0  img013-00999   img013     262.5  ...  2.575758              1          B\n",
              "1  img013-00985   img013      19.5  ...  1.177979              0          C\n",
              "2  img013-00983   img013      25.5  ...  1.240421              0          C\n",
              "3  img011-00856   img011     978.0  ...  1.669396              1          A\n",
              "4  img012-00847   img012    4432.5  ...  1.508786              0          B\n",
              "\n",
              "[5 rows x 19 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 130
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "bd151d5a-bf05-4e5a-9163-62e11697439f",
        "id": "12D7HZoRJh2w",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo Kmeans\n",
        "\n",
        "pd.crosstab(final_teste[\"ANOTACAO\"],final_teste[\"pred_y_kmeans\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>img011</th>\n",
              "      <td>68.780488</td>\n",
              "      <td>14.146341</td>\n",
              "      <td>17.073171</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img012</th>\n",
              "      <td>54.634146</td>\n",
              "      <td>7.317073</td>\n",
              "      <td>38.048780</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img013</th>\n",
              "      <td>64.390244</td>\n",
              "      <td>5.365854</td>\n",
              "      <td>30.243902</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_kmeans          0          1          2\n",
              "ANOTACAO                                      \n",
              "img011         68.780488  14.146341  17.073171\n",
              "img012         54.634146   7.317073  38.048780\n",
              "img013         64.390244   5.365854  30.243902"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 131
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "7b29635e-7b60-48b8-fe17-797a7f2eb7a0",
        "id": "EXtQk0pqJh20",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e a Random Forest\n",
        "pd.crosstab(final_teste[\"ANOTACAO\"],final_teste[\"pred_y_rf\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>A</th>\n",
              "      <th>B</th>\n",
              "      <th>C</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>img011</th>\n",
              "      <td>96.097561</td>\n",
              "      <td>1.951220</td>\n",
              "      <td>1.951220</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img012</th>\n",
              "      <td>3.414634</td>\n",
              "      <td>94.146341</td>\n",
              "      <td>2.439024</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img013</th>\n",
              "      <td>3.414634</td>\n",
              "      <td>1.951220</td>\n",
              "      <td>94.634146</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_rf          A          B          C\n",
              "ANOTACAO                                  \n",
              "img011     96.097561   1.951220   1.951220\n",
              "img012      3.414634  94.146341   2.439024\n",
              "img013      3.414634   1.951220  94.634146"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 132
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "8TBpcFriQjh9"
      },
      "source": [
        "Criando a base HOG (teste) e aplicando o K-means e random forest na mesma"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "85bPWYF0QjiM",
        "colab": {}
      },
      "source": [
        "# HOG\n",
        "base_hog = []\n",
        "\n",
        "for img in x_teste_esq:\n",
        "  fd, hog_image = hog(img, orientations=8, pixels_per_cell=(16, 16),\n",
        "                    cells_per_block=(1, 1), visualize=True, multichannel=False)\n",
        "  base_hog.append(hog_image)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "4769645a-78da-44f5-f7a9-ec174aa4b096",
        "id": "lqs74udTQjic",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 341
        }
      },
      "source": [
        "# Visualizando uma imagem com HOG\n",
        "\n",
        "plt.imshow(base_hog[150])"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "<matplotlib.image.AxesImage at 0x7f4a4fa431d0>"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 135
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 360x360 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "GhxVGR4jQjij",
        "colab": {}
      },
      "source": [
        "# Ajustando as dimensões da base para aplicação do k-means\n",
        "\n",
        "base_hog_ok = np.array(base_hog)\n",
        "nsamples, nx, ny = base_hog_ok.shape\n",
        "base_hog_final = base_hog_ok.reshape((nsamples,nx*ny))"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "QHUAE32eQjip",
        "colab": {}
      },
      "source": [
        "# Construindo os modelos na base HOG\n",
        "\n",
        "# K-means\n",
        "final_teste['pred_y_kmeans_hog'] = kmeans_hog.predict(base_hog_final)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "f0848a7e-618c-4500-bbdd-ab0e17497441",
        "id": "PXFKckgQQjiu",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_teste.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img013-00999</td>\n",
              "      <td>img013</td>\n",
              "      <td>262.5</td>\n",
              "      <td>68</td>\n",
              "      <td>95</td>\n",
              "      <td>6460</td>\n",
              "      <td>16.5</td>\n",
              "      <td>30.0</td>\n",
              "      <td>0.715789</td>\n",
              "      <td>792.535990</td>\n",
              "      <td>456.114785</td>\n",
              "      <td>0.040635</td>\n",
              "      <td>-1</td>\n",
              "      <td>2508.0</td>\n",
              "      <td>240.611816</td>\n",
              "      <td>0.527525</td>\n",
              "      <td>2.575758</td>\n",
              "      <td>1</td>\n",
              "      <td>B</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img013-00985</td>\n",
              "      <td>img013</td>\n",
              "      <td>19.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>9626.458311</td>\n",
              "      <td>433.261973</td>\n",
              "      <td>0.003121</td>\n",
              "      <td>1</td>\n",
              "      <td>5304.0</td>\n",
              "      <td>265.673660</td>\n",
              "      <td>0.613194</td>\n",
              "      <td>1.177979</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img013-00983</td>\n",
              "      <td>img013</td>\n",
              "      <td>25.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>8000.878151</td>\n",
              "      <td>451.688380</td>\n",
              "      <td>0.004081</td>\n",
              "      <td>1</td>\n",
              "      <td>5037.0</td>\n",
              "      <td>256.844223</td>\n",
              "      <td>0.568631</td>\n",
              "      <td>1.240421</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img011-00856</td>\n",
              "      <td>img011</td>\n",
              "      <td>978.0</td>\n",
              "      <td>64</td>\n",
              "      <td>86</td>\n",
              "      <td>5504</td>\n",
              "      <td>21.0</td>\n",
              "      <td>32.0</td>\n",
              "      <td>0.744186</td>\n",
              "      <td>144.967169</td>\n",
              "      <td>376.534051</td>\n",
              "      <td>0.177689</td>\n",
              "      <td>0</td>\n",
              "      <td>3297.0</td>\n",
              "      <td>243.667625</td>\n",
              "      <td>0.647133</td>\n",
              "      <td>1.669396</td>\n",
              "      <td>1</td>\n",
              "      <td>A</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00847</td>\n",
              "      <td>img012</td>\n",
              "      <td>4432.5</td>\n",
              "      <td>89</td>\n",
              "      <td>82</td>\n",
              "      <td>7298</td>\n",
              "      <td>23.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.085366</td>\n",
              "      <td>21.757949</td>\n",
              "      <td>310.551297</td>\n",
              "      <td>0.607358</td>\n",
              "      <td>-1</td>\n",
              "      <td>4837.0</td>\n",
              "      <td>280.695008</td>\n",
              "      <td>0.903860</td>\n",
              "      <td>1.508786</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  pred_y_kmeans  pred_y_rf  pred_y_kmeans_hog\n",
              "0  img013-00999   img013     262.5  ...              1          B                  0\n",
              "1  img013-00985   img013      19.5  ...              0          C                  1\n",
              "2  img013-00983   img013      25.5  ...              0          C                  1\n",
              "3  img011-00856   img011     978.0  ...              1          A                  0\n",
              "4  img012-00847   img012    4432.5  ...              0          B                  2\n",
              "\n",
              "[5 rows x 20 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 138
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "e0568de9-958c-47c1-fc20-b1764ea18a0a",
        "id": "g1n64_PVQjiz",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo Kmeans_hog\n",
        "\n",
        "pd.crosstab(final_teste[\"ANOTACAO\"],final_teste[\"pred_y_kmeans_hog\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "      <th>0</th>\n",
              "      <th>1</th>\n",
              "      <th>2</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>img011</th>\n",
              "      <td>99.512195</td>\n",
              "      <td>0.487805</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img012</th>\n",
              "      <td>16.585366</td>\n",
              "      <td>2.439024</td>\n",
              "      <td>80.975610</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img013</th>\n",
              "      <td>14.146341</td>\n",
              "      <td>85.365854</td>\n",
              "      <td>0.487805</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_kmeans_hog          0          1          2\n",
              "ANOTACAO                                          \n",
              "img011             99.512195   0.487805   0.000000\n",
              "img012             16.585366   2.439024  80.975610\n",
              "img013             14.146341  85.365854   0.487805"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 139
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "s6WyzysRQji7",
        "colab": {}
      },
      "source": [
        "# Predict na base HOG e já agregando a coluna ao final\n",
        "\n",
        "final_teste['pred_y_rf_hog'] = model_rf_hog.predict(base_hog_final)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "5abbf0d5-d3a6-4c01-a38d-6a0db6d01088",
        "id": "9RLlVfY3Qji_",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 226
        }
      },
      "source": [
        "# Validação\n",
        "\n",
        "final_teste.head()"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>IMAGEM</th>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th>AREA_OBJ</th>\n",
              "      <th>ALTURA</th>\n",
              "      <th>LARGURA</th>\n",
              "      <th>AREA_CONTORNO</th>\n",
              "      <th>BALAN_HOR</th>\n",
              "      <th>BALAN_VER</th>\n",
              "      <th>RAZAO</th>\n",
              "      <th>COMPACIDADE</th>\n",
              "      <th>PERIM_CONTORNO</th>\n",
              "      <th>RETANG.</th>\n",
              "      <th>EULER</th>\n",
              "      <th>AREA_FECHO</th>\n",
              "      <th>PERIM_FECHO</th>\n",
              "      <th>CONVEXIDADE</th>\n",
              "      <th>SOLIDEZ</th>\n",
              "      <th>pred_y_kmeans</th>\n",
              "      <th>pred_y_rf</th>\n",
              "      <th>pred_y_kmeans_hog</th>\n",
              "      <th>pred_y_rf_hog</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>img013-00999</td>\n",
              "      <td>img013</td>\n",
              "      <td>262.5</td>\n",
              "      <td>68</td>\n",
              "      <td>95</td>\n",
              "      <td>6460</td>\n",
              "      <td>16.5</td>\n",
              "      <td>30.0</td>\n",
              "      <td>0.715789</td>\n",
              "      <td>792.535990</td>\n",
              "      <td>456.114785</td>\n",
              "      <td>0.040635</td>\n",
              "      <td>-1</td>\n",
              "      <td>2508.0</td>\n",
              "      <td>240.611816</td>\n",
              "      <td>0.527525</td>\n",
              "      <td>2.575758</td>\n",
              "      <td>1</td>\n",
              "      <td>B</td>\n",
              "      <td>0</td>\n",
              "      <td>A</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>img013-00985</td>\n",
              "      <td>img013</td>\n",
              "      <td>19.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>9626.458311</td>\n",
              "      <td>433.261973</td>\n",
              "      <td>0.003121</td>\n",
              "      <td>1</td>\n",
              "      <td>5304.0</td>\n",
              "      <td>265.673660</td>\n",
              "      <td>0.613194</td>\n",
              "      <td>1.177979</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "      <td>1</td>\n",
              "      <td>C</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>img013-00983</td>\n",
              "      <td>img013</td>\n",
              "      <td>25.5</td>\n",
              "      <td>88</td>\n",
              "      <td>71</td>\n",
              "      <td>6248</td>\n",
              "      <td>28.5</td>\n",
              "      <td>20.0</td>\n",
              "      <td>1.239437</td>\n",
              "      <td>8000.878151</td>\n",
              "      <td>451.688380</td>\n",
              "      <td>0.004081</td>\n",
              "      <td>1</td>\n",
              "      <td>5037.0</td>\n",
              "      <td>256.844223</td>\n",
              "      <td>0.568631</td>\n",
              "      <td>1.240421</td>\n",
              "      <td>0</td>\n",
              "      <td>C</td>\n",
              "      <td>1</td>\n",
              "      <td>C</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>img011-00856</td>\n",
              "      <td>img011</td>\n",
              "      <td>978.0</td>\n",
              "      <td>64</td>\n",
              "      <td>86</td>\n",
              "      <td>5504</td>\n",
              "      <td>21.0</td>\n",
              "      <td>32.0</td>\n",
              "      <td>0.744186</td>\n",
              "      <td>144.967169</td>\n",
              "      <td>376.534051</td>\n",
              "      <td>0.177689</td>\n",
              "      <td>0</td>\n",
              "      <td>3297.0</td>\n",
              "      <td>243.667625</td>\n",
              "      <td>0.647133</td>\n",
              "      <td>1.669396</td>\n",
              "      <td>1</td>\n",
              "      <td>A</td>\n",
              "      <td>0</td>\n",
              "      <td>A</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>img012-00847</td>\n",
              "      <td>img012</td>\n",
              "      <td>4432.5</td>\n",
              "      <td>89</td>\n",
              "      <td>82</td>\n",
              "      <td>7298</td>\n",
              "      <td>23.0</td>\n",
              "      <td>19.5</td>\n",
              "      <td>1.085366</td>\n",
              "      <td>21.757949</td>\n",
              "      <td>310.551297</td>\n",
              "      <td>0.607358</td>\n",
              "      <td>-1</td>\n",
              "      <td>4837.0</td>\n",
              "      <td>280.695008</td>\n",
              "      <td>0.903860</td>\n",
              "      <td>1.508786</td>\n",
              "      <td>0</td>\n",
              "      <td>B</td>\n",
              "      <td>2</td>\n",
              "      <td>B</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "         IMAGEM ANOTACAO  AREA_OBJ  ...  pred_y_rf  pred_y_kmeans_hog  pred_y_rf_hog\n",
              "0  img013-00999   img013     262.5  ...          B                  0              A\n",
              "1  img013-00985   img013      19.5  ...          C                  1              C\n",
              "2  img013-00983   img013      25.5  ...          C                  1              C\n",
              "3  img011-00856   img011     978.0  ...          A                  0              A\n",
              "4  img012-00847   img012    4432.5  ...          B                  2              B\n",
              "\n",
              "[5 rows x 21 columns]"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 141
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "outputId": "14de2455-3d46-4419-9c0c-9659100c4ea2",
        "id": "ITYU4_RxQjjE",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 175
        }
      },
      "source": [
        "# Validação cruzada entre o rótulo e os grupos descobertos pelo RF_hog\n",
        "\n",
        "pd.crosstab(final_teste[\"ANOTACAO\"],final_teste[\"pred_y_rf_hog\"]).apply(lambda r: r/r.sum()*100, axis=1)"
      ],
      "execution_count": 0,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/html": [
              "<div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th>pred_y_rf_hog</th>\n",
              "      <th>A</th>\n",
              "      <th>B</th>\n",
              "      <th>C</th>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>ANOTACAO</th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "      <th></th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>img011</th>\n",
              "      <td>98.536585</td>\n",
              "      <td>1.463415</td>\n",
              "      <td>0.000000</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img012</th>\n",
              "      <td>3.902439</td>\n",
              "      <td>95.121951</td>\n",
              "      <td>0.975610</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>img013</th>\n",
              "      <td>3.902439</td>\n",
              "      <td>0.000000</td>\n",
              "      <td>96.097561</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>"
            ],
            "text/plain": [
              "pred_y_rf_hog          A          B          C\n",
              "ANOTACAO                                      \n",
              "img011         98.536585   1.463415   0.000000\n",
              "img012          3.902439  95.121951   0.975610\n",
              "img013          3.902439   0.000000  96.097561"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 142
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "n-BwMbC4R3oN",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Exportando as bases finais\n",
        "\n",
        "final_treino.to_csv(r'/content/base_final_treino.csv')\n",
        "final_teste.to_csv(r'/content/base_final_teste.csv')"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "pOjF8-QgRQqN",
        "colab_type": "text"
      },
      "source": [
        "**CONCLUSÃO**\n",
        "\n",
        "Ao final deste trabalho, podemos concluir que os modelos que utilizam HOG performaram de forma muito superior os modelos com os descritores extraídos separadamente. Isso ocorreu tanto para o modelo k-means quanto para o Random Forest.\n",
        "\n",
        "Vale ressaltar que o modelo de árvores performou melhor quando comparado ao k-means, tanto para a versão de descritores extraídos separadamente quanto para o HOG."
      ]
    }
  ]
}
