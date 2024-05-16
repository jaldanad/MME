#Identificación de palabras clave en una búsqueda referencial en Pubmed utilizando expresiones regulares.

!pip install python-docx
from google.colab import files
import re
from docx import Document

uploaded = files.upload()

# Función para leer el contenido del archivo de Word
def leer_docx(file_path):
    doc = Document(file_path)
    texto = []
    for para in doc.paragraphs:
        texto.append(para.text)
    return '\n'.join(texto)

# Cargar el archivo de Word
file_path = 'MME_REFERENCIAS.docx'
texto = leer_docx(file_path)

# Definir las palabras clave
palabras_clave_primarias = [
    "Machine learning", "statistical learning", "random forest", "PCA features", 
    "ANN", "Electronic Health Records", "Learning-based", "feature engineering", 
    "Natural language processing", "cluster analysis", "Risk Prediction Model", 
    "Interpretable Predictive Models"
]

palabras_clave_secundarias = [
    "obstetric", "maternal", "maternal health", "Severe Maternal Morbidity", 
    "maternal risk", "pre-eclampsia", "Preeclampsia", "bleeding", "gestations",
    "pregnancy", "pregnancies", "postpartum"
]

# Convertir todas las palabras clave a minúsculas para hacer una búsqueda sin distinción de mayúsculas
palabras_clave_primarias = [palabra.lower() for palabra in palabras_clave_primarias]
palabras_clave_secundarias = [palabra.lower() for palabra in palabras_clave_secundarias]

# Función para buscar palabras clave en el texto y extraer títulos, año y mes
def buscar_palabras_clave(texto, palabras_primarias, palabras_secundarias):
    titulos_relevantes = []
    referencias = re.split(r'\n\s*\d{1,3}:\s+', texto)
    for ref in referencias:
        ref_lower = ref.lower()
        for palabra_primaria in palabras_primarias:
            if palabra_primaria in ref_lower:
                if any(palabra_secundaria in ref_lower for palabra_secundaria in palabras_secundarias):
                    # Extraer el título basado en la estructura del archivo
                    match = re.search(r'^\s*(.*?)\.\s+(\d{4}\s+\w+)\s+', ref, re.DOTALL)
                    if match:
                        titulo = match.group(1).strip()
                        fecha = match.group(2).strip()
                        titulos_relevantes.append((titulo, fecha))
                    break
    return titulos_relevantes

# Función para contar el número de referencias
def contar_referencias(texto):
    return len(re.findall(r'\b\d{1,3}:\s', texto))

# Buscar los títulos relevantes
titulos_relevantes = buscar_palabras_clave(texto, palabras_clave_primarias, palabras_clave_secundarias)

# Contar el número de referencias
num_referencias = contar_referencias(texto)

# Numerar y mostrar los títulos relevantes con año y mes
for i, (titulo, fecha) in enumerate(titulos_relevantes, 1):
    print(f"{i}. {titulo} ({fecha})")

# Mostrar el número de referencias
print(f"Número total de referencias: {num_referencias}")
