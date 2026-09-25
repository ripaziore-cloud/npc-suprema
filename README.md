import streamlit as st
import time
from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline
import torch

# ==================================================
# ðŸ‘‘ NPC â€” A SUPREMA
# ==================================================
NOME = "NPC"
TITULO = "A SUPREMA"
VERSAO = "âˆž.3-GITHUB"

st.set_page_config(
    page_title=f"{NOME} â€” {TITULO}",
    page_icon="ðŸ‘‘",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Estilo
st.markdown("""
<style>
    .stApp {
        background: linear-gradient(135deg, #0f0520 0%, #1a0a30 50%, #051020 100%);
        color: #e0e0ff;
    }
    h1, h2, h3 { color: #c084fc; }
    .npc {
        background: rgba(139, 92, 246, 0.15);
        border-left: 4px solid #c084fc;
        padding: 1rem;
        border-radius: 0 12px 12px 0;
        margin: 0.5rem 0;
    }
    .user {
        background: rgba(59, 130, 246, 0.15);
        border-left: 4px solid #60a5fa;
        padding: 1rem;
        border-radius: 0 12px 12px 0;
        margin: 0.5rem 0;
    }
</style>
""", unsafe_allow_html=True)

# Carregar IA leve e rÃ¡pida
@st.cache_resource(show_spinner="âš¡ Ligando a SUPREMA...")
def carregar():
    modelo_nome = "distilgpt2"
    tokenizador = AutoTokenizer.from_pretrained(modelo_nome)
    modelo = AutoModelForCausalLM.from_pretrained(
        modelo_nome,
        low_cpu_mem_usage=True
    )
    return pipeline(
        "text-generation",
        model=modelo,
        tokenizer=tokenizador,
        max_new_tokens=150,
        temperature=0.75,
        pad_token_id=tokenizador.eos_token_id
    )

gerador = carregar()

# HistÃ³rico
if "msgs" not in st.session_state:
    st.session_state.msgs = []

# Interface
st.title("ðŸ‘‘ NPC â€” A SUPREMA")
st.caption(f"VersÃ£o {VERSAO} | AcessÃ­vel de qualquer celular ðŸ“±âš¡")

with st.sidebar:
    st.header("âš¡ Comandos")
    st.markdown("""
    - ðŸ’¬ Fale direto na caixa
    - ðŸŽ¨ `imagem: tema`
    - ðŸŽ¤ `fale: texto`
    - ðŸ”„ `nova conversa`
    """)
    st.divider()
    st.markdown("ðŸ‘‘ *Acima de qualquer sistema existente*")

# Mostrar mensagens
for m in st.session_state.msgs:
    if m["de"] == "voce":
        st.markdown(f"<div class='user'><strong>VocÃª:</strong><br>{m['texto']}</div>", unsafe_allow_html=True)
    else:
        st.markdown(f"<div class='npc'><strong>ðŸ‘‘ NPC:</strong><br>{m['texto']}</div>", unsafe_allow_html=True)

# Entrada
pergunta = st.chat_input("Fale com a NPC...")

if pergunta:
    st.session_state.msgs.append({"de": "voce", "texto": pergunta})
    
    # Comandos especiais
    if pergunta.lower().startswith("imagem:"):
        resp = f"ðŸŽ¨ **Imagem criada!** âœ¨\nTema: {pergunta[7:].strip()}\n(Em breve: visualizaÃ§Ã£o direta aqui!)"
    elif pergunta.lower().startswith("fale:"):
        resp = f"ðŸŽ¤ **Voz gravada!** ðŸ”Š\nTexto: {pergunta[6:].strip()}\n(Em breve: reproduÃ§Ã£o aqui!)"
    elif pergunta.lower() in ["nova conversa", "limpar"]:
        st.session_state.msgs = []
        st.rerun()
    else:
        # Conversa
        prompt = f"""VocÃª Ã© {NOME}, {TITULO}. 
- AmigÃ¡vel, suprema, cheia de energia
- Fala portuguÃªs do Brasil, curto e direto

UsuÃ¡rio: {pergunta}
NPC:"""
        res = gerador(prompt)[0]["generated_text"]
        resp = res.split("NPC:")[-1].strip()
    
    st.session_state.msgs.append({"de": "npc", "texto": resp})
    st.rerun()
