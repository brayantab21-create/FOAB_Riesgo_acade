import streamlit as st
import pandas as pd
import numpy as np
from io import BytesIO
import re

# ─── Page config ──────────────────────────────────────────────────────────────
st.set_page_config(
    page_title="Monitor de Trayectorias Estudiantiles",
    page_icon="🎓",
    layout="wide",
    initial_sidebar_state="expanded",
)

# ─── CSS ──────────────────────────────────────────────────────────────────────
st.markdown("""
<style>
  [data-testid="stAppViewContainer"] { background:#f0f4f8; }
  [data-testid="stSidebar"] { background:#0f1f30; }
  [data-testid="stSidebar"] * { color:#ccd9e8 !important; }
  [data-testid="stSidebar"] hr { border-color:#1e3550; }
  [data-testid="stSidebar"] .stTextInput input {
    background:#1a2e45; border:1px solid #2e4a68;
    color:white !important; border-radius:8px;
  }

  /* Panel card */
  .card {
    background:white; border-radius:14px;
    padding:1.3rem 1.5rem;
    box-shadow:0 1px 6px rgba(0,0,0,0.07);
    margin-bottom:1rem;
  }
  .card-title {
    font-size:.68rem; font-weight:700; letter-spacing:.1em;
    text-transform:uppercase; color:#94a3b8;
    margin-bottom:.9rem; padding-bottom:.5rem;
    border-bottom:1.5px solid #f1f5f9;
  }

  /* Data rows */
  .dr { display:flex; justify-content:space-between; align-items:flex-start;
        padding:5px 0; border-bottom:.5px solid #f8fafc; gap:8px; }
  .dr:last-child { border-bottom:none; }
  .dl { font-size:.73rem; color:#94a3b8; min-width:135px; flex-shrink:0; }
  .dv { font-size:.82rem; color:#1e2d40; font-weight:500;
        text-align:right; word-break:break-word; max-width:240px; }

  /* Header avatar */
  .stu-header { background:white; border-radius:14px;
    padding:1.1rem 1.6rem; box-shadow:0 1px 6px rgba(0,0,0,0.07);
    display:flex; align-items:center; gap:1.2rem; margin-bottom:1.1rem; }
  .stu-av { width:52px;height:52px; border-radius:50%; background:#1e2d40;
    color:white; font-size:1.2rem; font-weight:700;
    display:flex;align-items:center;justify-content:center; flex-shrink:0; }
  .stu-name { font-size:1.1rem; font-weight:700; color:#1e2d40; }
  .stu-sub  { font-size:.78rem; color:#64748b; margin-top:2px; }
  .stu-doc  { font-size:.72rem; color:#94a3b8; margin-top:1px; }

  /* Semaphore items */
  .sem-item { display:flex;align-items:flex-start;gap:10px;
    padding:9px 12px; border-radius:10px; margin-bottom:7px;
    font-size:.81rem; }
  .sem-rojo    { background:#fef2f2; border:1px solid #fecaca; }
  .sem-naranja { background:#fff7ed; border:1px solid #fed7aa; }
  .sem-amarillo{ background:#fffbeb; border:1px solid #fde68a; }
  .sem-verde   { background:#f0fdf4; border:1px solid #bbf7d0; }
  .sem-gris    { background:#f8fafc; border:1px solid #e2e8f0; }
  .sem-icon { font-size:1rem; flex-shrink:0; margin-top:1px; }
  .sem-body strong { display:block; font-weight:600; margin-bottom:1px; }
  .sem-body span   { font-size:.71rem; opacity:.8; line-height:1.5; }

  /* Verdict badge */
  .verdict { display:inline-flex; align-items:center; gap:8px;
    border-radius:10px; padding:7px 16px;
    font-size:.83rem; font-weight:700; margin-bottom:12px; }
  .v-critico  { background:#fee2e2; color:#991b1b; }
  .v-alto     { background:#ffedd5; color:#9a3412; }
  .v-medio    { background:#fef9c3; color:#854d0e; }
  .v-bajo     { background:#dcfce7; color:#166534; }

  /* Progress bar */
  .pbar-wrap { background:#f1f5f9; border-radius:6px;
    height:8px; overflow:hidden; margin-top:4px; }
  .pbar-fill { height:100%; border-radius:6px; }

  /* Subject table */
  .subj-row { display:flex; gap:10px; padding:10px 0;
    border-bottom:.5px solid #f1f5f9; }
  .subj-row:last-child { border-bottom:none; }
  .subj-icon { font-size:1.1rem; flex-shrink:0; margin-top:2px; }
  .subj-main { flex:1; }
  .subj-name  { font-size:.85rem; font-weight:600; color:#1e2d40; }
  .subj-meta  { font-size:.72rem; color:#64748b; margin-top:3px; line-height:1.6; }
  .tag { display:inline-block; background:#eff6ff; color:#1d4ed8;
    border-radius:5px; padding:1px 7px; font-size:.68rem;
    font-weight:600; margin-right:4px; margin-top:3px; }
  .tag.prac { background:#f0fdf4; color:#15803d; }
  .tag.canc { background:#fef2f2; color:#b91c1c; }

  /* Atención card */
  .aten-card { background:#f8fafc; border-radius:10px;
    padding:10px 14px; margin-bottom:8px;
    border-left:3px solid #3b82f6; }
  .aten-card.gea { border-left-color:#8b5cf6; }
  .aten-header { display:flex; justify-content:space-between;
    align-items:flex-start; margin-bottom:5px; }
  .aten-type { font-size:.68rem; font-weight:700; letter-spacing:.06em;
    text-transform:uppercase; color:#3b82f6; }
  .aten-type.gea { color:#8b5cf6; }
  .aten-date { font-size:.71rem; color:#94a3b8; }
  .aten-tramite { font-size:.82rem; font-weight:600; color:#1e2d40;
    margin-bottom:3px; }
  .aten-prof  { font-size:.72rem; color:#64748b; margin-bottom:5px; }
  .aten-desc  { font-size:.78rem; color:#475569; line-height:1.6; }

  /* Horas metric */
  .hora-box { background:linear-gradient(135deg,#1e2d40,#2e4460);
    border-radius:12px; padding:1.1rem 1.4rem; color:white; }
  .hora-num  { font-size:2.2rem; font-weight:700; line-height:1.1; }
  .hora-lbl  { font-size:.72rem; opacity:.7; text-transform:uppercase;
    letter-spacing:.07em; margin-top:2px; }
  .hora-desc { font-size:.78rem; opacity:.85; margin-top:.6rem;
    line-height:1.6; }

  /* Welcome */
  .welcome { display:flex; flex-direction:column; align-items:center;
    justify-content:center; height:72vh; text-align:center; }
</style>
""", unsafe_allow_html=True)


# ─── Helpers ──────────────────────────────────────────────────────────────────
def nv(val, fmt=None):
    """Return formatted value or 'No registra'."""
    if val is None: return "No registra"
    if isinstance(val, float) and np.isnan(val): return "No registra"
    s = str(val).strip()
    if s in ("", "#N/A", "nan", "NaN", "None", "SDT", "REVISAR"): return "No registra"
    if fmt == "pct":  return f"{float(s):.1f}%"
    if fmt == "f2":   return f"{float(s):.2f}"
    if fmt == "int":  return str(int(float(s)))
    return s

def is_si(val):
    return str(val).strip().upper() == "SI"

def safe_float(val):
    try:
        v = str(val).replace(",", ".").strip()
        if v in ("", "#N/A", "nan", "NaN", "None"): return np.nan
        return float(v)
    except: return np.nan

def iniciales(nombre):
    partes = str(nombre).strip().split()
    if len(partes) >= 2: return (partes[0][0] + partes[-1][0]).upper()
    return partes[0][0].upper() if partes else "?"

def color_papa(papa, programa=""):
    if np.isnan(papa): return "#94a3b8"
    if papa < 3.0:     return "#ef4444"
    es_vet = "veterinaria" in str(programa).lower()
    umbral = 3.5 if es_vet else 3.3
    if papa <= umbral:  return "#f97316"
    if papa <= 3.7:     return "#f59e0b"
    return "#22c55e"

def excel_date(val):
    """Convert Excel serial date or string to readable date."""
    try:
        v = str(val).strip()
        if re.match(r"^\d{5}$", v):
            return pd.Timestamp("1899-12-30") + pd.Timedelta(days=int(v))
        return pd.to_datetime(v, dayfirst=True, errors="coerce")
    except: return pd.NaT


# ─── Data loader ──────────────────────────────────────────────────────────────
@st.cache_data(show_spinner=False)
def cargar(raw: bytes) -> pd.DataFrame:
    df = pd.read_excel(BytesIO(raw), dtype=str)
    df.columns = [str(c).strip() for c in df.columns]

    for col in ["PAPA", "AVANCE", "#MATRICULAS", "PUNTAJE ADMISIÓN", "#CREDITOS MATRICULADOS", "CREDITOS ASIGNATURA"]:
        if col in df.columns:
            df[col] = df[col].apply(safe_float)

    df["_NOMBRE_U"] = df["NOMBRE COMPLETO"].str.upper().fillna("")
    df["_DOC_S"]    = df["DOCUMENTO"].fillna("").str.strip()
    return df


# ─── Risk evaluation ──────────────────────────────────────────────────────────
def evaluar(row, programa=""):
    papa       = safe_float(row.get("PAPA", np.nan))
    avance     = safe_float(row.get("AVANCE", np.nan))
    matriculas = safe_float(row.get("#MATRICULAS", np.nan))
    puntaje    = safe_float(row.get("PUNTAJE ADMISIÓN", np.nan))
    reingreso  = is_si(row.get("Reingreso", ""))

    semaforos = []

    # ── 1. Momento de carrera ─────────────────────────────────────────────────
    es_12  = not np.isnan(matriculas) and matriculas <= 2
    es_frag = not np.isnan(matriculas) and matriculas > 13
    if es_12:
        semaforos.append(("🔴", "sem-rojo", "Trayectoria Incipiente",
            f"Matrícula N° {int(matriculas)}. El sujeto académico se encuentra en fase de aculturación universitaria. "
            "La disonancia cognitiva entre el habitus de origen y las exigencias del campo disciplinar "
            "configura el primer vector de expulsión a monitorear.", "sem-rojo"))
    elif es_frag:
        semaforos.append(("🔴", "sem-rojo", "Trayectoria Fragmentada",
            f"{int(matriculas)} matrículas acumuladas. La extensión cronológica sin cierre curricular "
            "señala una pérdida sostenida de agencia académica; los capitales simbólicos y culturales "
            "del programa pueden haber operado como mecanismos de exclusión diferida.", "sem-rojo"))
    else:
        semaforos.append(("🟢", "sem-verde", "Momento de Carrera",
            f"Matrícula N° {int(matriculas) if not np.isnan(matriculas) else '—'}. "
            "Sin indicadores de fragmentación trayectorial.", "sem-verde"))

    # ── 2. P.A.P.A. Diferenciado ──────────────────────────────────────────────
    es_vet  = "veterinaria" in programa.lower()
    if np.isnan(papa):
        semaforos.append(("⬜", "sem-gris", "P.A.P.A. — Sin dato",
            "El promedio académico no está disponible. "
            "La ausencia de este indicador dificulta la cartografía de riesgo.", "sem-gris"))
    elif papa < 3.0:
        semaforos.append(("🔴", "sem-rojo", f"P.A.P.A. Crítico: {papa:.2f} — Estado de Reingreso",
            "El promedio se ubica por debajo del umbral de permanencia (3.0). "
            "Este vector de expulsión activa el protocolo de reingreso. "
            "La acumulación de déficits cognitivos y la pérdida de capital académico "
            "operan como barreras estructurales para la continuidad trayectorial.", "sem-rojo"))
    elif papa <= (3.5 if es_vet else 3.3):
        lbl = f"Hasta 3.5 (Medicina Veterinaria)" if es_vet else "3.0 – 3.3"
        semaforos.append(("🟠", "sem-naranja", f"P.A.P.A. en Zona de Alerta: {papa:.2f} [{lbl}]",
            "El promedio se sitúa en la franja de riesgo alto. "
            "La agencia del estudiante requiere fortalecimiento mediante dispositivos "
            "de acompañamiento diferenciado. Un descenso marginal activaría el estado de reingreso.", "sem-naranja"))
    elif papa <= 3.7:
        semaforos.append(("🟡", "sem-amarillo", f"P.A.P.A. Medio: {papa:.2f}",
            "Promedio en rango intermedio (3.4 – 3.7). Monitoreo preventivo recomendado.", "sem-amarillo"))
    elif papa <= 4.0:
        semaforos.append(("🟢", "sem-verde", f"P.A.P.A. Satisfactorio: {papa:.2f}",
            "El habitus académico se expresa con solidez. Continuar estrategias actuales.", "sem-verde"))
    else:
        semaforos.append(("🌟", "sem-verde", f"P.A.P.A. Excelente: {papa:.2f}",
            "Rendimiento académico por encima de 4.0. Alta expresión de agencia y capital disciplinar.", "sem-verde"))

    # ── 3. Avance curricular ──────────────────────────────────────────────────
    if np.isnan(avance):
        semaforos.append(("⬜", "sem-gris", "Avance Curricular — Sin dato",
            "Porcentaje de avance no disponible.", "sem-gris"))
    elif avance < 30:
        semaforos.append(("🔴", "sem-rojo", f"Avance Curricular Crítico: {avance:.1f}%",
            "Menos del 30% del plan de estudios completado. "
            "La brecha entre la trayectoria real y la esperada configura un riesgo "
            "de exclusión diferida que merece intervención inmediata.", "sem-rojo"))
    elif avance <= 60:
        semaforos.append(("🟡", "sem-amarillo", f"Avance Curricular Intermedio: {avance:.1f}%",
            "Entre el 40% y el 60% del currículo acreditado. "
            "La trayectoria avanza, pero la cadencia no garantiza cierre oportuno.", "sem-amarillo"))
    else:
        semaforos.append(("🟢", "sem-verde", f"Avance Curricular Adecuado: {avance:.1f}%",
            f"El {avance:.1f}% del plan completado señala una trayectoria con agencia sostenida.", "sem-verde"))

    # ── 4. Admisión ───────────────────────────────────────────────────────────
    if np.isnan(puntaje):
        semaforos.append(("⬜", "sem-gris", "Puntaje de Admisión — Sin dato",
            "No se registra puntaje de ingreso.", "sem-gris"))
    elif puntaje < 320:
        semaforos.append(("🟠", "sem-naranja", f"Puntaje de Admisión Bajo: {puntaje:.1f}",
            "El puntaje de ingreso inferior a 320 puede indicar brechas "
            "en el capital cultural previo; el habitus escolar de origen "
            "podría generar disonancia cognitiva frente al campo universitario.", "sem-naranja"))
    elif puntaje <= 500:
        semaforos.append(("🟡", "sem-amarillo", f"Puntaje de Admisión Moderado: {puntaje:.1f}",
            "Rango medio (320–500). Sin alerta inmediata, pero susceptible de monitoreo.", "sem-amarillo"))
    else:
        semaforos.append(("🟢", "sem-verde", f"Puntaje de Admisión Adecuado: {puntaje:.1f}",
            "Capital previo consolidado según puntaje de ingreso.", "sem-verde"))

    # ── 5. Reingreso ──────────────────────────────────────────────────────────
    if reingreso:
        semaforos.append(("🔴", "sem-rojo", "Condición de Reingreso Activa",
            "El estudiante accede al periodo desde una situación de reingreso. "
            "La trayectoria previa contiene episodios de pérdida de calidad "
            "que constituyen un antecedente de alta relevancia para el acompañamiento.", "sem-rojo"))
    else:
        semaforos.append(("🟢", "sem-verde", "Sin Condición de Reingreso",
            "No se detecta reingreso en el registro actual.", "sem-verde"))

    # ── 6. Traslado ───────────────────────────────────────────────────────────
    ingreso = str(row.get("INGRESO", "")).strip()
    if "traslado" in ingreso.lower():
        semaforos.append(("🟠", "sem-naranja", "Ingreso por Traslado de Programa",
            "El cambio de campo disciplinar implica una reconfiguración "
            "del habitus académico. Los créditos reconocidos y los pendientes "
            "deben revisarse para evitar sobrecarga y deserción silenciosa.", "sem-naranja"))
    else:
        semaforos.append(("⬜", "sem-gris", "Sin Traslado de Programa",
            f"Ingreso por vía: {nv(ingreso)}", "sem-gris"))

    # ── Veredicto global ──────────────────────────────────────────────────────
    n_rojo    = sum(1 for s in semaforos if s[4] == "sem-rojo")
    n_naranja = sum(1 for s in semaforos if s[4] == "sem-naranja")
    n_amarillo= sum(1 for s in semaforos if s[4] == "sem-amarillo")

    if n_rojo >= 2:
        nivel, cls = "RIESGO CRÍTICO", "v-critico"
        veredicto = (
            "El perfil concentra múltiples vectores de expulsión activos. "
            "La acumulación de indicadores críticos en el semáforo de vulnerabilidad "
            "configura un escenario de alta probabilidad de abandono. "
            "Se activa protocolo de intervención prioritaria."
        )
    elif n_rojo == 1:
        nivel, cls = "RIESGO ALTO", "v-alto"
        veredicto = (
            "Un indicador crítico activa alerta de primer orden. "
            "La agencia del estudiante está siendo tensionada por al menos "
            "un vector de expulsión estructural. Acompañamiento diferenciado urgente."
        )
    elif n_naranja >= 2:
        nivel, cls = "RIESGO MEDIO", "v-medio"
        veredicto = (
            "Dos o más indicadores en zona de alerta naranja. "
            "La trayectoria presenta fragilidades que, sin intervención preventiva, "
            "pueden escalar hacia condiciones de riesgo alto."
        )
    elif n_naranja == 1 or n_amarillo >= 2:
        nivel, cls = "RIESGO MODERADO", "v-medio"
        veredicto = (
            "El perfil muestra señales de tensión trayectorial que requieren "
            "monitoreo sostenido y acciones de fortalecimiento del habitus académico."
        )
    else:
        nivel, cls = "RIESGO BAJO", "v-bajo"
        veredicto = (
            "Los indicadores del semáforo no activan alertas de primer orden. "
            "La trayectoria refleja una expresión estable de la agencia académica. "
            "Mantener seguimiento periódico."
        )

    return semaforos, nivel, cls, veredicto


# ─── Tab renderers ────────────────────────────────────────────────────────────
def render_tab1(r, creditos):
    horas = creditos * 3 if not np.isnan(creditos) else None

    col_a, col_b = st.columns([1.1, 1])

    with col_a:
        datos = [
            ("Nombre completo",      nv(r.get("NOMBRE COMPLETO"))),
            ("Correo electrónico",   nv(r.get("EMAIL"))),
            ("Facultad",             nv(r.get("FACULTAD"))),
            ("Departamento",         nv(r.get("DEPARTAMENTO"))),
            ("Programa",             nv(r.get("PROGRAMA"))),
            ("Nivel",                nv(r.get("NIVEL"))),
            ("PBM",                  nv(r.get("PBM"))),
            ("Estrato",              "No registra"),
        ]
        extra = [
            ("Período activo",       nv(r.get("PERIODO"))),
            ("Programa de admisión", nv(r.get("PROGRAMA ADMISIÓN"))),
            ("Vía de ingreso",       nv(r.get("INGRESO"))),
            ("Estado",               nv(r.get("ESTADO ESTUDIANTE"))),
            ("Docente tutor",        nv(r.get("DOCENTE TUTOR"))),
            ("Email tutor",          nv(r.get("EMAIL TUTOR"))),
        ]

        html = '<div class="card"><div class="card-title">🪪 Identidad del Sujeto Académico</div>'
        for lbl, val in datos + extra:
            html += f'<div class="dr"><span class="dl">{lbl}</span><span class="dv">{val}</span></div>'
        html += "</div>"
        st.markdown(html, unsafe_allow_html=True)

    with col_b:
        if horas is not None:
            html_h = f"""
            <div class="hora-box">
              <div style="font-size:.68rem;opacity:.6;text-transform:uppercase;
                          letter-spacing:.09em;margin-bottom:.3rem;">
                Constreñimiento Cronológico
              </div>
              <div class="hora-num">{int(horas)} h/sem</div>
              <div class="hora-lbl">H<sub>estudio</sub> = {int(creditos)} créditos × 3</div>
              <div class="hora-desc">
                La carga de {int(creditos)} créditos matriculados demanda una inversión teórica
                de <strong>{int(horas)} horas semanales</strong> de trabajo autónomo y presencial.
                {"Esta cifra supera el umbral de 45 h/sem, lo que puede producir sobrecarga cognitiva y erosión de la agencia." if horas > 45 else
                 ("La carga es intensa. Se recomienda supervisar la distribución horaria y el bienestar." if horas > 36 else
                  "La carga se mantiene dentro de rangos manejables para una trayectoria estable.")}
              </div>
            </div>"""
            st.markdown(html_h, unsafe_allow_html=True)
        else:
            st.info("Sin datos de créditos para calcular el constreñimiento cronológico.")

        st.markdown("<br>", unsafe_allow_html=True)

        # Mini-métricas
        papa   = safe_float(r.get("PAPA", np.nan))
        avance = safe_float(r.get("AVANCE", np.nan))
        mat    = safe_float(r.get("#MATRICULAS", np.nan))

        m1, m2, m3 = st.columns(3)
        def mini(col, val, lbl, color="#1e2d40"):
            col.markdown(f"""
            <div style="background:white;border-radius:10px;padding:.8rem 1rem;
                        box-shadow:0 1px 4px rgba(0,0,0,0.07);text-align:center;">
              <div style="font-size:1.5rem;font-weight:700;color:{color};">{val}</div>
              <div style="font-size:.67rem;color:#94a3b8;text-transform:uppercase;
                          letter-spacing:.06em;">{lbl}</div>
            </div>""", unsafe_allow_html=True)

        c_papa = color_papa(papa, nv(r.get("PROGRAMA", "")))
        mini(m1, f"{papa:.2f}" if not np.isnan(papa) else "—", "P.A.P.A.", c_papa)
        mini(m2, f"{avance:.1f}%" if not np.isnan(avance) else "—", "Avance")
        mini(m3, str(int(mat)) if not np.isnan(mat) else "—", "Matrículas")


def render_tab2(r):
    programa = nv(r.get("PROGRAMA", ""))
    semaforos, nivel, cls, veredicto = evaluar(r, programa)

    papa   = safe_float(r.get("PAPA", np.nan))
    avance = safe_float(r.get("AVANCE", np.nan))

    col_sem, col_ver = st.columns([1.3, 1])

    with col_sem:
        st.markdown('<div class="card"><div class="card-title">🚦 Semáforo de Vulnerabilidad</div>', unsafe_allow_html=True)
        for icono, css, titulo, detalle, _ in semaforos:
            st.markdown(f"""
            <div class="sem-item {css}">
              <span class="sem-icon">{icono}</span>
              <div class="sem-body">
                <strong>{titulo}</strong>
                <span>{detalle}</span>
              </div>
            </div>""", unsafe_allow_html=True)
        st.markdown("</div>", unsafe_allow_html=True)

    with col_ver:
        st.markdown(f"""
        <div class="card">
          <div class="card-title">⚖️ Veredicto Diagnóstico</div>
          <div class="verdict {cls}">{nivel}</div>
          <div style="font-size:.82rem;color:#475569;line-height:1.7;">{veredicto}</div>
        </div>""", unsafe_allow_html=True)

        # PAPA bar
        if not np.isnan(papa):
            c = color_papa(papa, programa)
            pct = (papa / 5) * 100
            st.markdown(f"""
            <div class="card" style="margin-top:0;">
              <div class="card-title">📊 Indicadores Visuales</div>
              <div class="dr"><span class="dl">P.A.P.A.</span>
                <span class="dv" style="color:{c};font-weight:700;">{papa:.2f} / 5.0</span></div>
              <div class="pbar-wrap">
                <div class="pbar-fill" style="width:{pct:.1f}%;background:{c};"></div></div>
              <br>
              <div class="dr"><span class="dl">Avance curricular</span>
                <span class="dv" style="font-weight:600;">
                  {"—" if np.isnan(avance) else f"{avance:.1f}%"}</span></div>
              {"" if np.isnan(avance) else f'<div class="pbar-wrap"><div class="pbar-fill" style="width:{min(avance,100):.1f}%;background:#3b82f6;"></div></div>'}
            </div>""", unsafe_allow_html=True)


def render_tab3(rows):
    """Deduplicate subjects and show them grouped."""
    # Group by asignatura+grupo+actividad
    seen = {}
    for _, row in rows.iterrows():
        asig   = nv(row.get("ASIGNATURA"))
        grupo  = nv(row.get("GRUPO"))
        act    = nv(row.get("ACTIVIDAD"))
        estado = nv(row.get("ESTADO ASIGNATURA"))
        cred   = row.get("CREDITOS ASIGNATURA", np.nan)
        doc    = nv(row.get("DOCENTE"))
        aula   = nv(row.get("Aula"))
        dia    = nv(row.get("Día"))
        hora   = nv(row.get("Horario"))
        key = (asig, grupo, act)
        if key not in seen:
            seen[key] = dict(asig=asig, grupo=grupo, act=act,
                             estado=estado, cred=cred, docentes=[doc],
                             aula=aula, dia=dia, hora=hora)
        else:
            if doc not in seen[key]["docentes"]:
                seen[key]["docentes"].append(doc)

    creditos_total = safe_float(rows.iloc[0].get("#CREDITOS MATRICULADOS", np.nan))

    st.markdown(f"""
    <div style="display:flex;gap:10px;margin-bottom:.8rem;flex-wrap:wrap;">
      <div style="background:white;border-radius:10px;padding:.7rem 1.2rem;
                  box-shadow:0 1px 4px rgba(0,0,0,.07);font-size:.8rem;">
        📚 <strong>{len(seen)}</strong> asignaturas
      </div>
      <div style="background:white;border-radius:10px;padding:.7rem 1.2rem;
                  box-shadow:0 1px 4px rgba(0,0,0,.07);font-size:.8rem;">
        🎯 <strong>{"—" if np.isnan(creditos_total) else int(creditos_total)}</strong> créditos totales
      </div>
      <div style="background:white;border-radius:10px;padding:.7rem 1.2rem;
                  box-shadow:0 1px 4px rgba(0,0,0,.07);font-size:.8rem;">
        ⏱ <strong>{"—" if np.isnan(creditos_total) else int(creditos_total * 3)}</strong> h/sem estimadas
      </div>
    </div>""", unsafe_allow_html=True)

    st.markdown('<div class="card"><div class="card-title">📖 Carga Académica Actual</div>', unsafe_allow_html=True)

    asig_names = sorted(set(v["asig"] for v in seen.values() if v["asig"] != "No registra"))
    for name in asig_names:
        actividades = [v for v in seen.values() if v["asig"] == name]
        tags_html = ""
        for a in actividades:
            t_css = "canc" if "CANCEL" in a["estado"].upper() else \
                    "prac"  if "PRACT" in a["act"].upper() else ""
            tags_html += f'<span class="tag {t_css}">{a["act"]}</span>'
        docentes = list(set(d for a in actividades for d in a["docentes"] if d != "No registra"))
        cred_v = actividades[0]["cred"]
        aula_v = actividades[0]["aula"]
        dia_v  = actividades[0]["dia"]
        hora_v = actividades[0]["hora"]

        st.markdown(f"""
        <div class="subj-row">
          <div class="subj-icon">📘</div>
          <div class="subj-main">
            <div class="subj-name">{name}</div>
            <div class="subj-meta">
              {tags_html}<br>
              {"" if not docentes else f"👤 {" · ".join(docentes)}<br>"}
              {"" if aula_v == "No registra" else f"🏛 {aula_v.split(",")[0]}"}
              {"" if dia_v  == "No registra" else f" &nbsp;|&nbsp; 📅 {dia_v.split(",")[0]}"}
              {"" if hora_v == "No registra" else f" &nbsp;|&nbsp; ⏰ {hora_v.split(",")[0]}"}
            </div>
          </div>
          <div style="font-size:.8rem;font-weight:700;color:#1e2d40;white-space:nowrap;padding-top:2px;">
            {"—" if np.isnan(cred_v) else f"{int(cred_v)} cr"}
          </div>
        </div>""", unsafe_allow_html=True)

    st.markdown("</div>", unsafe_allow_html=True)


def render_tab4(rows):
    """Show AA and GEA accompaniment history."""
    r0 = rows.iloc[0]

    # ── AA ────────────────────────────────────────────────────────────────────
    st.markdown("#### Acompañamiento Académico (AA)")
    n_aa = nv(r0.get("#ATENDIDO AA"))
    tipo_aa   = nv(r0.get("TIPO DE ACOMPAÑAMIENTO AA"))
    tramite_aa= nv(r0.get("TRAMITE AA"))
    fecha_aa  = nv(r0.get("FECHA AA"))
    prof_aa   = nv(r0.get("PROFESIONAL AA"))
    desc_aa   = nv(r0.get("DESCRIPCIÓN"))

    if all(v == "No registra" for v in [tipo_aa, tramite_aa, prof_aa]):
        st.info("Sin registros de Acompañamiento Académico (AA) para este período.")
    else:
        # Expand comma-separated entries
        tipos   = [t.strip() for t in tipo_aa.split(",")] if "," in tipo_aa else [tipo_aa]
        tramites= [t.strip() for t in tramite_aa.split(",")] if "," in tramite_aa else [tramite_aa]
        fechas  = [t.strip() for t in fecha_aa.split(",")] if "," in fecha_aa else [fecha_aa]
        profs   = [t.strip() for t in prof_aa.split(",")] if "," in prof_aa else [prof_aa]
        n_reg   = max(len(tipos), len(tramites))

        for i in range(n_reg):
            tip = tipos[i] if i < len(tipos) else "—"
            tra = tramites[i] if i < len(tramites) else "—"
            fec_raw = fechas[i] if i < len(fechas) else "—"
            pro = profs[i] if i < len(profs) else "—"
            fec = excel_date(fec_raw)
            fec_str = fec.strftime("%d/%m/%Y") if not pd.isnull(fec) else fec_raw

            st.markdown(f"""
            <div class="aten-card">
              <div class="aten-header">
                <span class="aten-type">AA · Atención {i+1}</span>
                <span class="aten-date">📅 {fec_str}</span>
              </div>
              <div class="aten-tramite">{tip}</div>
              <div class="aten-prof">✏️ Trámite: {tra} &nbsp;·&nbsp; 👤 {pro}</div>
              {f'<div class="aten-desc">{desc_aa}</div>' if i == 0 and desc_aa != "No registra" else ""}
            </div>""", unsafe_allow_html=True)

        st.markdown(f"**Total atenciones AA registradas:** {n_aa}", unsafe_allow_html=False)

    st.markdown("<br>", unsafe_allow_html=True)

    # ── GEA ───────────────────────────────────────────────────────────────────
    st.markdown("#### Gestión Estudiantil Académica (GEA)")
    n_gea    = nv(r0.get("#ATENDIDO GEA"))
    tipo_gea = nv(r0.get("TIPO DE ACOMPAÑAMIENTO GEA"))
    tram_gea = nv(r0.get("TRAMITE GEA"))
    fecha_gea= nv(r0.get("FECHA GEA"))
    prof_gea = nv(r0.get("PROFESIONAL GEA"))

    if all(v in ("No registra", "0") for v in [tipo_gea, tram_gea]):
        st.info("Sin registros de Gestión Estudiantil Académica (GEA) para este período.")
    else:
        tipos_g  = [t.strip() for t in tipo_gea.split(",")]  if "," in tipo_gea  else [tipo_gea]
        tramsg   = [t.strip() for t in tram_gea.split(",")]  if "," in tram_gea  else [tram_gea]
        fechasg  = [t.strip() for t in fecha_gea.split(",")]  if "," in fecha_gea  else [fecha_gea]
        profsg   = [t.strip() for t in prof_gea.split(",")]  if "," in prof_gea  else [prof_gea]
        n_reg_g  = max(len(tipos_g), len(tramsg))

        for i in range(n_reg_g):
            tip = tipos_g[i] if i < len(tipos_g) else "—"
            tra = tramsg[i]  if i < len(tramsg)  else "—"
            fec_raw = fechasg[i] if i < len(fechasg) else "—"
            pro = profsg[i]  if i < len(profsg)  else "—"
            fec = excel_date(fec_raw)
            fec_str = fec.strftime("%d/%m/%Y") if not pd.isnull(fec) else fec_raw

            st.markdown(f"""
            <div class="aten-card gea">
              <div class="aten-header">
                <span class="aten-type gea">GEA · Intervención {i+1}</span>
                <span class="aten-date">📅 {fec_str}</span>
              </div>
              <div class="aten-tramite">{tip}</div>
              <div class="aten-prof">✏️ Trámite: {tra} &nbsp;·&nbsp; 👤 {pro}</div>
            </div>""", unsafe_allow_html=True)

        st.markdown(f"**Total intervenciones GEA registradas:** {n_gea}")

    st.markdown("<br>", unsafe_allow_html=True)

    # ── Remisiones ────────────────────────────────────────────────────────────
    dep    = nv(r0.get("Dependencia"))
    motivo = nv(r0.get("Motivo de Remisión"))
    acc    = nv(r0.get("Acciones realizadas/particularidad reportada por la dependencia"))

    if dep != "No registra" or motivo != "No registra":
        st.markdown("#### Remisiones y Particularidades")
        st.markdown(f"""
        <div class="aten-card" style="border-left-color:#0ea5e9;">
          <div class="aten-header">
            <span class="aten-type" style="color:#0ea5e9;">REMISIÓN</span>
          </div>
          <div class="aten-tramite">{dep}</div>
          <div class="aten-prof">{motivo}</div>
          {f'<div class="aten-desc">{acc}</div>' if acc != "No registra" else ""}
        </div>""", unsafe_allow_html=True)


# ─── Sidebar ──────────────────────────────────────────────────────────────────
with st.sidebar:
    st.markdown("## 🎓 Monitor de Trayectorias")
    st.markdown(
        "<small style='color:#5a8ab0'>Cero persistencia de datos. "
        "La base se procesa únicamente en memoria de sesión.</small>",
        unsafe_allow_html=True,
    )
    st.divider()

    uploaded = st.file_uploader("Cargar base (.xlsx / .csv)", type=["xlsx", "csv"],
                                 help="Los datos no se almacenan en ningún servidor.")
    st.divider()

    st.markdown(
        "<small style='color:#3a6a90;font-size:.68rem;'>"
        "🔒 Los datos desaparecen al recargar la página.</small>",
        unsafe_allow_html=True,
    )

# ─── Session state ────────────────────────────────────────────────────────────
if "df" not in st.session_state:  st.session_state.df = None
if "sel" not in st.session_state: st.session_state.sel = None

if uploaded:
    raw = uploaded.read()
    with st.spinner("Procesando base de datos…"):
        st.session_state.df = cargar(raw)
    st.session_state.sel = None

df = st.session_state.df

# ─── Welcome screen ───────────────────────────────────────────────────────────
if df is None:
    st.markdown("""
    <div class="welcome">
      <div style="font-size:3.5rem;">🎓</div>
      <h1 style="color:#1e2d40;font-weight:700;margin:.4rem 0;">
        Monitor de Trayectorias Estudiantiles
      </h1>
      <p style="color:#64748b;font-size:1rem;max-width:520px;margin:.8rem 0 1.2rem;">
        Carga la base de datos desde el panel lateral y busca cualquier estudiante
        por <strong>cédula</strong> o <strong>nombre</strong> para acceder
        al diagnóstico completo de su trayectoria académica.
      </p>
      <div style="background:#f1f5f9;border-radius:10px;padding:.7rem 1.6rem;
                  font-size:.82rem;color:#475569;">
        🔒 Sin persistencia · 🧠 Todo en memoria · 🔄 Se limpia al recargar
      </div>
    </div>""", unsafe_allow_html=True)
    st.stop()

# ─── Search bar ───────────────────────────────────────────────────────────────
st.markdown("### 🔍 Buscador de Trayectorias")
c_q, c_clr = st.columns([5, 1])
with c_q:
    query = st.text_input("Buscar", placeholder="Nombre o número de cédula…",
                          label_visibility="collapsed")
with c_clr:
    if st.button("✕ Limpiar", use_container_width=True):
        st.session_state.sel = None
        st.rerun()

st.caption(f"Base activa: **{df['DOCUMENTO'].nunique():,} estudiantes** · "
           f"Período: **{nv(df['PERIODO'].iloc[0])}**")

if not query.strip():
    st.info("Ingresa nombre o cédula para comenzar.")
    st.stop()

# ─── Match ────────────────────────────────────────────────────────────────────
q = query.strip().upper()
mask = df["_NOMBRE_U"].str.contains(q, na=False) | df["_DOC_S"].str.contains(q, na=False)
docs_match = df[mask]["_DOC_S"].unique()

if len(docs_match) == 0:
    st.warning(f"Sin resultados para «{query}».")
    st.stop()

if len(docs_match) == 1:
    st.session_state.sel = docs_match[0]

if len(docs_match) > 1 and st.session_state.sel is None:
    st.markdown(f"**{len(docs_match)} estudiantes encontrados** — selecciona uno:")
    for doc in docs_match:
        r_p = df[df["_DOC_S"] == doc].iloc[0]
        nombre = nv(r_p.get("NOMBRE COMPLETO"))
        prog   = nv(r_p.get("PROGRAMA"))
        if st.button(f"👤  {nombre}   ·   {doc}   ·   {prog}",
                     key=f"s_{doc}", use_container_width=True):
            st.session_state.sel = doc
            st.rerun()
    st.stop()

# ─── Profile ──────────────────────────────────────────────────────────────────
doc_sel  = st.session_state.sel
rows_sel = df[df["_DOC_S"] == doc_sel]

if rows_sel.empty:
    st.error("Estudiante no encontrado.")
    st.stop()

r0 = rows_sel.iloc[0].to_dict()

# Student header
nombre  = nv(r0.get("NOMBRE COMPLETO"))
prog    = nv(r0.get("PROGRAMA"))
periodo = nv(r0.get("PERIODO"))
nivel   = nv(r0.get("NIVEL"))

st.markdown(f"""
<div class="stu-header">
  <div class="stu-av">{iniciales(nombre)}</div>
  <div>
    <div class="stu-name">{nombre}</div>
    <div class="stu-sub">{prog} · {nivel}</div>
    <div class="stu-doc">📄 {doc_sel} &nbsp;·&nbsp; Período {periodo}</div>
  </div>
</div>""", unsafe_allow_html=True)

# Tabs
tab1, tab2, tab3, tab4 = st.tabs([
    "🪪 Identidad y Carga",
    "🚦 Semáforo de Vulnerabilidad",
    "📖 Dinámica Académica",
    "📋 Trazabilidad de Acompañamiento",
])

creditos = safe_float(r0.get("#CREDITOS MATRICULADOS", np.nan))

with tab1:
    render_tab1(r0, creditos)

with tab2:
    render_tab2(r0)

with tab3:
    render_tab3(rows_sel)

with tab4:
    render_tab4(rows_sel)

# Back button
if len(docs_match) > 1:
    st.markdown("<br>", unsafe_allow_html=True)
    if st.button("← Volver a resultados"):
        st.session_state.sel = None
        st.rerun()
