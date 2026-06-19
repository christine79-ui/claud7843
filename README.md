# claud7843
import React, { useState, useEffect, useCallback, useMemo } from 'react';
import { Users, FileText, ClipboardList, BarChart3, GitCompare, Plus, ChevronRight, Loader2, Copy, Check, ChevronDown, Save, X, AlertTriangle, CheckCircle2, RefreshCcw } from 'lucide-react';

// ===== constants.js =====

// 9개 영역 (업무수행일지 / 급여제공결과평가지 공통)
const NINE_AREAS = [
  { key: 'physical', label: '신체활동지원', hint: '식사, 이동, 배설, 개인위생 등' },
  { key: 'cognitive', label: '인지지원', hint: '인지활동 프로그램 제공, 기억력 관리 등' },
  { key: 'emotional', label: '정서지원', hint: '말벗, 위로, 상담 및 지지' },
  { key: 'daily', label: '일상생활지원', hint: '청소 및 주변정돈, 세탁, 취사' },
  { key: 'personal', label: '개인활동지원', hint: '외출 동행, 일상업무 대행' },
  { key: 'rehab', label: '기능회복훈련', hint: '물리치료, 재활 운동 및 훈련 등' },
  { key: 'safety', label: '환경 및 안전관리', hint: '낙상/욕창 방지 등 안전한 환경 조성' },
  { key: 'health', label: '건강관리/간호', hint: '투약 관리, 활력징후 측정, 병원 진료 동행 등' },
  { key: 'etc', label: '기타(자유기재)', hint: '위 8개 영역 외 특이사항' },
];

const STATUS_OPTIONS = ['유지', '호전', '악화'];

// 욕구조사기록지 8개 영역
const NEEDS_AREAS = [
  { key: 'health', label: '1. 건강상태', hint: '보유질환, 복약 및 의료이용, 구강과 영양상태' },
  { key: 'adl', label: '2. 일상생활기능', hint: '위생관리, 일상생활, 도구적 일상생활, 배뇨·배변기능 및 방법' },
  { key: 'rehab', label: '3. 재활 및 신체기능', hint: '근골격계 증상, 보행상태, 낙상, 신체기능' },
  { key: 'nursing', label: '4. 간호관리', hint: '호흡기간호, 피부간호, 소화기간호, 통증간호, 배뇨·배변간호, 내분비간호' },
  { key: 'cognition', label: '5. 인지 및 의사소통', hint: '인지기능, 행동증상, 심리증상, 의사소통(이해력/표현력/시력/청력)' },
  { key: 'support', label: '6. 가족 및 지지체계', hint: '주거상태, 지지체계, 사회적교류, 지역사회자원' },
  { key: 'environment', label: '7. 거주환경', hint: '층수, 생활환경(엘리베이터, 계단, 장애물, 바닥, 냉난방, 조명, 주방, 화장실 등)' },
  { key: 'wishes', label: '8. 희망서비스', hint: '신체활동지원, 일상생활지원, 기능회복훈련, 인지관리 및 정서지원, 건강 및 간호관리, 방문목욕, 복지용구 등' },
];

const NEEDS_TYPES = ['최초', '정기', '상태변화', '기타'];

// storage 헬퍼
export async function safeGet(key, shared = false) {
  try {
    const res = await window.storage.get(key, shared);
    return res ? res.value : null;
  } catch (e) {
    return null;
  }
}

export async function safeSet(key, value, shared = false) {
  try {
    const res = await window.storage.set(key, value, shared);
    return !!res;
  } catch (e) {
    return false;
  }
}

export async function safeList(prefix, shared = false) {
  try {
    const res = await window.storage.list(prefix, shared);
    return res ? res.keys : [];
  } catch (e) {
    return [];
  }
}

export async function safeDelete(key, shared = false) {
  try {
    await window.storage.delete(key, shared);
    return true;
  } catch (e) {
    return false;
  }
}

function todayStr() {
  const d = new Date();
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
}

function monthStr(date) {
  const d = date ? new Date(date) : new Date();
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}`;
}


// ===== Common.jsx =====


function SectionCard({ title, hint, children, defaultOpen = true }) {
  const [open, setOpen] = useState(defaultOpen);
  return (
    <div style={s.card}>
      <div style={s.cardHead} onClick={() => setOpen(!open)}>
        <div>
          <div style={s.cardTitle}>{title}</div>
          {hint && <div style={s.cardHint}>{hint}</div>}
        </div>
        <ChevronDown size={16} color="#A39B8B" style={{ transform: open ? 'rotate(180deg)' : 'none', transition: 'transform .15s' }} />
      </div>
      {open && <div style={s.cardBody}>{children}</div>}
    </div>
  );
}

function StatusPicker({ value, onChange, options }) {
  return (
    <div style={s.statusRow}>
      {options.map((opt) => (
        <button
          key={opt}
          onClick={() => onChange(opt)}
          style={{
            ...s.statusBtn,
            ...(value === opt ? statusActiveStyle(opt) : {}),
          }}
        >
          {opt}
        </button>
      ))}
    </div>
  );
}

function statusActiveStyle(opt) {
  if (opt === '유지') return { background: '#EAF1ED', color: '#1F4D3D', borderColor: '#1F4D3D' };
  if (opt === '호전') return { background: '#EAF3FB', color: '#1A5276', borderColor: '#1A5276' };
  return { background: '#FBEDEA', color: '#A3401F', borderColor: '#A3401F' };
}

function TextArea({ value, onChange, placeholder, minRows = 2 }) {
  return (
    <textarea
      value={value || ''}
      onChange={(e) => onChange(e.target.value)}
      placeholder={placeholder}
      rows={minRows}
      style={s.textarea}
    />
  );
}

function TextField({ value, onChange, placeholder, style }) {
  return (
    <input
      value={value || ''}
      onChange={(e) => onChange(e.target.value)}
      placeholder={placeholder}
      style={{ ...s.input, ...style }}
    />
  );
}

function OutputBox({ text, label = '복사용 텍스트' }) {
  const [copied, setCopied] = useState(false);
  const doCopy = async () => {
    try {
      await navigator.clipboard.writeText(text);
      setCopied(true);
      setTimeout(() => setCopied(false), 1600);
    } catch (e) {
      const ta = document.createElement('textarea');
      ta.value = text;
      document.body.appendChild(ta);
      ta.select();
      document.execCommand('copy');
      document.body.removeChild(ta);
      setCopied(true);
      setTimeout(() => setCopied(false), 1600);
    }
  };
  return (
    <div style={s.outputWrap}>
      <div style={s.outputHead}>
        <span style={s.outputLabel}>{label}</span>
        <button style={{ ...s.copyBtn, ...(copied ? s.copyBtnDone : {}) }} onClick={doCopy}>
          {copied ? <Check size={13} /> : <Copy size={13} />}
          {copied ? '복사됨' : '복사'}
        </button>
      </div>
      <pre style={s.outputPre}>{text}</pre>
    </div>
  );
}

function SaveBar({ onSave, savedAt, saving }) {
  return (
    <div style={s.saveBar}>
      <span style={s.saveAt}>{saving ? '저장 중...' : savedAt ? `마지막 저장: ${savedAt}` : '아직 저장 안 됨'}</span>
      <button style={s.saveBtn} onClick={onSave}>
        <Save size={14} /> 저장
      </button>
    </div>
  );
}

function PasteImportBox({ onImport }) {
  const [open, setOpen] = useState(false);
  const [raw, setRaw] = useState('');
  if (!open) {
    return (
      <button style={s.importToggle} onClick={() => setOpen(true)}>
        지난 기록 붙여넣기로 불러오기
      </button>
    );
  }
  return (
    <div style={s.importBox}>
      <div style={s.cardHint}>이전에 작성했던 텍스트를 붙여넣으면 참고용으로 표시됩니다. 항목별 자동 매칭은 되지 않으니 직접 보며 옮겨 입력해 주세요.</div>
      <textarea
        value={raw}
        onChange={(e) => setRaw(e.target.value)}
        placeholder="여기에 지난달 기록 텍스트를 붙여넣기..."
        rows={6}
        style={s.textarea}
      />
      <div style={{ display: 'flex', gap: 8, marginTop: 8 }}>
        <button style={s.modalCancel} onClick={() => setOpen(false)}>닫기</button>
        <button
          style={s.modalConfirm}
          onClick={() => {
            onImport(raw);
            setOpen(false);
          }}
        >
          참고창에 표시
        </button>
      </div>
    </div>
  );
}

function ReferencePanel({ text, onClear }) {
  if (!text) return null;
  return (
    <div style={s.refPanel}>
      <div style={s.refHead}>
        <span style={{ fontSize: 12, fontWeight: 700, color: '#8A7A4A' }}>참고: 붙여넣은 이전 기록</span>
        <button style={s.refClear} onClick={onClear}>닫기</button>
      </div>
      <pre style={s.refPre}>{text}</pre>
    </div>
  );
}

const s = {
  card: {
    background: '#FFFFFF',
    border: '1px solid #EDE7DA',
    borderRadius: 10,
    marginBottom: 10,
    overflow: 'hidden',
  },
  cardHead: {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: '12px 14px',
    cursor: 'pointer',
  },
  cardTitle: { fontSize: 14, fontWeight: 700, color: '#2E2A24' },
  cardHint: { fontSize: 11.5, color: '#A39B8B', marginTop: 2 },
  cardBody: { padding: '0 14px 14px', display: 'flex', flexDirection: 'column', gap: 10 },
  statusRow: { display: 'flex', gap: 6 },
  statusBtn: {
    flex: 1,
    padding: '7px 0',
    borderRadius: 7,
    border: '1px solid #E0DACE',
    background: '#FAF8F4',
    color: '#8A8275',
    fontSize: 12.5,
    fontWeight: 600,
    cursor: 'pointer',
  },
  textarea: {
    width: '100%',
    padding: '9px 11px',
    borderRadius: 8,
    border: '1px solid #DDD5C7',
    fontSize: 13.5,
    lineHeight: 1.55,
    outline: 'none',
    resize: 'vertical',
    fontFamily: 'inherit',
    background: '#FFFDF9',
  },
  input: {
    width: '100%',
    padding: '9px 11px',
    borderRadius: 8,
    border: '1px solid #DDD5C7',
    fontSize: 13.5,
    outline: 'none',
    background: '#FFFDF9',
  },
  outputWrap: {
    border: '1px solid #D9CDA8',
    borderRadius: 10,
    background: '#FFFCF3',
    marginTop: 4,
  },
  outputHead: {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: '9px 12px',
    borderBottom: '1px solid #EFE5C8',
  },
  outputLabel: { fontSize: 12, fontWeight: 700, color: '#8A7A4A' },
  copyBtn: {
    display: 'flex',
    alignItems: 'center',
    gap: 5,
    padding: '5px 10px',
    borderRadius: 6,
    border: '1px solid #D9CDA8',
    background: '#FFFFFF',
    fontSize: 11.5,
    fontWeight: 600,
    color: '#8A7A4A',
    cursor: 'pointer',
  },
  copyBtnDone: { background: '#EAF1ED', color: '#1F4D3D', borderColor: '#1F4D3D' },
  outputPre: {
    margin: 0,
    padding: '12px 14px',
    fontSize: 13,
    lineHeight: 1.75,
    whiteSpace: 'pre-wrap',
    wordBreak: 'break-word',
    fontFamily: "'Pretendard', 'Apple SD Gothic Neo', sans-serif",
    maxHeight: 360,
    overflowY: 'auto',
  },
  saveBar: {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: '10px 2px',
  },
  saveAt: { fontSize: 11.5, color: '#A39B8B' },
  saveBtn: {
    display: 'flex',
    alignItems: 'center',
    gap: 6,
    padding: '8px 16px',
    borderRadius: 8,
    border: 'none',
    background: '#1F4D3D',
    color: '#FFF',
    fontSize: 13,
    fontWeight: 700,
    cursor: 'pointer',
  },
  importToggle: {
    width: '100%',
    padding: '9px 0',
    borderRadius: 8,
    border: '1px dashed #D2C8B4',
    background: 'transparent',
    color: '#8A7A4A',
    fontSize: 12.5,
    fontWeight: 600,
    cursor: 'pointer',
    marginBottom: 10,
  },
  importBox: {
    border: '1px solid #E0DACE',
    borderRadius: 10,
    padding: 12,
    marginBottom: 10,
    background: '#FFFDF9',
  },
  modalCancel: {
    padding: '7px 13px',
    borderRadius: 7,
    border: '1px solid #DDD5C7',
    background: '#FFF',
    fontSize: 12.5,
    cursor: 'pointer',
  },
  modalConfirm: {
    padding: '7px 13px',
    borderRadius: 7,
    border: 'none',
    background: '#1F4D3D',
    color: '#FFF',
    fontSize: 12.5,
    fontWeight: 600,
    cursor: 'pointer',
  },
  refPanel: {
    border: '1px solid #EFE5C8',
    background: '#FFFCF3',
    borderRadius: 10,
    marginBottom: 12,
  },
  refHead: {
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: '8px 12px',
    borderBottom: '1px solid #F3EAD0',
  },
  refClear: {
    fontSize: 11,
    color: '#A3401F',
    background: 'transparent',
    border: 'none',
    cursor: 'pointer',
  },
  refPre: {
    margin: 0,
    padding: '10px 12px',
    fontSize: 12.5,
    lineHeight: 1.6,
    whiteSpace: 'pre-wrap',
    color: '#6B6354',
    maxHeight: 220,
    overflowY: 'auto',
  },
};

const sharedStyles = s;


// ===== WorkLog.jsx =====


const wlEmptyAreas = () =>
  NINE_AREAS.reduce((acc, a) => {
    acc[a.key] = { status: '유지', reason: '' };
    return acc;
  }, {});

function wlEmptyForm() {
  return {
    month: monthStr(),
    visitDate: '',
    needs: '',
    areas: wlEmptyAreas(),
    overview: '',
    planCheck: '',
    guardianConsult: '',
    futurePlan: '',
    savedAt: null,
  };
}

function WorkLog({ recipient }) {
  const [form, setForm] = useState(wlEmptyForm());
  const [refText, setRefText] = useState('');
  const [saving, setSaving] = useState(false);
  const [history, setHistory] = useState([]);
  const storageKey = `worklog:${recipient.id}:${form.month}`;

  const loadMonth = useCallback(
    async (month) => {
      const key = `worklog:${recipient.id}:${month}`;
      const raw = await safeGet(key, true);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          setForm({ ...wlEmptyForm(), ...parsed, areas: { ...wlEmptyAreas(), ...(parsed.areas || {}) }, month });
        } catch (e) {
          setForm({ ...wlEmptyForm(), month });
        }
      } else {
        setForm({ ...wlEmptyForm(), month });
      }
    },
    [recipient.id]
  );

  const loadHistory = useCallback(async () => {
    const keys = await safeList(`worklog:${recipient.id}:`, true);
    setHistory(keys.map((k) => k.split(':')[2]).sort().reverse());
  }, [recipient.id]);

  useEffect(() => {
    loadMonth(monthStr());
    loadHistory();
  }, [recipient.id, loadMonth, loadHistory]);

  const updateArea = (key, field, value) => {
    setForm((f) => ({ ...f, areas: { ...f.areas, [key]: { ...f.areas[key], [field]: value } } }));
  };

  const save = async () => {
    setSaving(true);
    const savedAt = new Date().toLocaleString('ko-KR');
    const toSave = { ...form, savedAt };
    await safeSet(storageKey, JSON.stringify(toSave), true);
    setForm(toSave);
    await loadHistory();
    setSaving(false);
  };

  const output = useMemo(() => buildWorkLogText(recipient, form), [recipient, form]);

  return (
    <div>
      <SectionCard title="대상 월 / 방문일" defaultOpen={true}>
        <div style={{ display: 'flex', gap: 8 }}>
          <select
            value={form.month}
            onChange={(e) => loadMonth(e.target.value)}
            style={{ flex: 1, padding: '9px 10px', borderRadius: 8, border: '1px solid #DDD5C7', fontSize: 13.5, background: '#FFFDF9' }}
          >
            {[monthStr(), ...history.filter((h) => h !== monthStr())].map((m) => (
              <option key={m} value={m}>{m}</option>
            ))}
          </select>
          <TextField value={form.visitDate} onChange={(v) => setForm((f) => ({ ...f, visitDate: v }))} placeholder="방문일자 (예: 2026-06-18)" style={{ flex: 1 }} />
        </div>
      </SectionCard>

      <PasteImportBox onImport={setRefText} />
      <ReferencePanel text={refText} onClear={() => setRefText('')} />

      <SectionCard title="욕구조사" hint="이번 방문 시 확인된 수급자·보호자의 욕구·요청사항">
        <TextArea value={form.needs} onChange={(v) => setForm((f) => ({ ...f, needs: v }))} placeholder="예: 보호자가 야간 배뇨 횟수 증가를 호소하며 기저귀 사용 검토를 요청함" />
      </SectionCard>

      <SectionCard title="심신상태 및 환경변화 (9개 영역)" hint="변화 없으면 '유지'만 선택, 변화 있으면 판단근거에 구체적 사실 기재">
        {NINE_AREAS.map((a) => (
          <div key={a.key} style={wlAreaBoxStyle}>
            <div style={wlAreaTitleRow}>
              <div>
                <div style={{ fontSize: 13.5, fontWeight: 700 }}>{a.label}</div>
                <div style={{ fontSize: 11, color: '#A39B8B' }}>{a.hint}</div>
              </div>
              <StatusPicker value={form.areas[a.key]?.status} onChange={(v) => updateArea(a.key, 'status', v)} options={STATUS_OPTIONS} />
            </div>
            {form.areas[a.key]?.status !== '유지' && (
              <TextArea
                value={form.areas[a.key]?.reason}
                onChange={(v) => updateArea(a.key, 'reason', v)}
                placeholder="판단근거: 병원명, 진단명, 처방약, 행동 변화 등 구체적 사실"
                minRows={2}
              />
            )}
          </div>
        ))}
      </SectionCard>

      <SectionCard title="종합의견">
        <TextArea value={form.overview} onChange={(v) => setForm((f) => ({ ...f, overview: v }))} placeholder="이번달 전반적 상태에 대한 종합 의견" />
      </SectionCard>

      <SectionCard title="급여제공계획 확인">
        <TextArea value={form.planCheck} onChange={(v) => setForm((f) => ({ ...f, planCheck: v }))} placeholder="예: 기존 급여제공계획대로 서비스 제공 중이며 별도 변경사항 없음" />
      </SectionCard>

      <SectionCard title="보호자 상담">
        <TextArea value={form.guardianConsult} onChange={(v) => setForm((f) => ({ ...f, guardianConsult: v }))} placeholder="보호자와 나눈 상담 내용" />
      </SectionCard>

      <SectionCard title="향후계획">
        <TextArea value={form.futurePlan} onChange={(v) => setForm((f) => ({ ...f, futurePlan: v }))} placeholder="예: 다음달 정기방문 시 배뇨양상 재확인 예정임" />
      </SectionCard>

      <SaveBar onSave={save} savedAt={form.savedAt} saving={saving} />
      <OutputBox text={output} label={`업무수행일지 — ${form.month}`} />
    </div>
  );
}

function buildWorkLogText(recipient, form) {
  const lines = [];
  lines.push(`[업무수행일지]`);
  lines.push(`수급자: ${recipient.name}`);
  lines.push(`대상월: ${form.month}${form.visitDate ? ` / 방문일: ${form.visitDate}` : ''}`);
  lines.push('');
  lines.push('■ 욕구조사');
  lines.push(form.needs?.trim() || '특이 욕구 확인되지 않음');
  lines.push('');
  lines.push('■ 심신상태 및 환경변화');
  NINE_AREAS.forEach((a) => {
    const area = form.areas[a.key] || {};
    const status = area.status || '유지';
    lines.push(`- ${a.label}: ${status}${status !== '유지' && area.reason?.trim() ? ` (${area.reason.trim()})` : ''}`);
  });
  lines.push('');
  lines.push('■ 종합의견');
  lines.push(form.overview?.trim() || '특이사항 없음');
  lines.push('');
  lines.push('■ 급여제공계획 확인');
  lines.push(form.planCheck?.trim() || '기존 계획대로 유지함');
  lines.push('');
  lines.push('■ 보호자상담');
  lines.push(form.guardianConsult?.trim() || '특이 상담내용 없음');
  lines.push('');
  lines.push('■ 향후계획');
  lines.push(form.futurePlan?.trim() || '기존 계획 유지하며 정기방문 지속할 예정임');
  return lines.join('\n');
}

const wlAreaBoxStyle = {
  border: '1px solid #EFE9DC',
  borderRadius: 8,
  padding: 10,
  display: 'flex',
  flexDirection: 'column',
  gap: 8,
  background: '#FCFAF6',
};

const wlAreaTitleRow = {
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'space-between',
  gap: 10,
};


// ===== NeedsAssessment.jsx =====


function naEmptyForm() {
  return {
    type: '정기',
    date: todayStr(),
    areas: NEEDS_AREAS.reduce((acc, a) => {
      acc[a.key] = '';
      return acc;
    }, {}),
    overview: '',
    savedAt: null,
  };
}

function NeedsAssessment({ recipient }) {
  const [form, setForm] = useState(naEmptyForm());
  const [refText, setRefText] = useState('');
  const [saving, setSaving] = useState(false);
  const [records, setRecords] = useState([]);
  const [activeKey, setActiveKey] = useState(null);

  const loadRecords = useCallback(async () => {
    const keys = await safeList(`needs:${recipient.id}:`, true);
    const sorted = keys.sort().reverse();
    setRecords(sorted);
  }, [recipient.id]);

  useEffect(() => {
    loadRecords();
    setForm(naEmptyForm());
    setActiveKey(null);
  }, [recipient.id, loadRecords]);

  const loadRecord = async (key) => {
    const raw = await safeGet(key, true);
    if (raw) {
      try {
        const parsed = JSON.parse(raw);
        setForm({ ...naEmptyForm(), ...parsed, areas: { ...naEmptyForm().areas, ...(parsed.areas || {}) } });
        setActiveKey(key);
      } catch (e) {}
    }
  };

  const startNew = () => {
    setForm(naEmptyForm());
    setActiveKey(null);
  };

  const save = async () => {
    setSaving(true);
    const savedAt = new Date().toLocaleString('ko-KR');
    const key = activeKey || `needs:${recipient.id}:${form.date}_${Date.now()}`;
    const toSave = { ...form, savedAt };
    await safeSet(key, JSON.stringify(toSave), true);
    setForm(toSave);
    setActiveKey(key);
    await loadRecords();
    setSaving(false);
  };

  const updateArea = (key, value) => {
    setForm((f) => ({ ...f, areas: { ...f.areas, [key]: value } }));
  };

  const output = useMemo(() => buildNeedsText(recipient, form), [recipient, form]);

  return (
    <div>
      <SectionCard title="작성일자 / 조사유형" defaultOpen={true}>
        <div style={{ display: 'flex', gap: 8, marginBottom: 8 }}>
          <button onClick={startNew} style={naNewBtnStyle}>+ 새 욕구조사 작성</button>
        </div>
        {records.length > 0 && (
          <select
            value={activeKey || ''}
            onChange={(e) => (e.target.value ? loadRecord(e.target.value) : startNew())}
            style={{ width: '100%', padding: '9px 10px', borderRadius: 8, border: '1px solid #DDD5C7', fontSize: 13, background: '#FFFDF9', marginBottom: 8 }}
          >
            <option value="">(작성 중 — 새 기록)</option>
            {records.map((k) => (
              <option key={k} value={k}>{k.split(':')[2]?.split('_')[0]}</option>
            ))}
          </select>
        )}
        <div style={{ display: 'flex', gap: 8 }}>
          <TextField value={form.date} onChange={(v) => setForm((f) => ({ ...f, date: v }))} placeholder="작성일 (YYYY-MM-DD)" style={{ flex: 1 }} />
        </div>
        <div style={{ display: 'flex', gap: 6, marginTop: 8 }}>
          {NEEDS_TYPES.map((t) => (
            <button
              key={t}
              onClick={() => setForm((f) => ({ ...f, type: t }))}
              style={{ ...naTypeBtnStyle, ...(form.type === t ? naTypeBtnActive : {}) }}
            >
              {t}
            </button>
          ))}
        </div>
      </SectionCard>

      <PasteImportBox onImport={setRefText} />
      <ReferencePanel text={refText} onClear={() => setRefText('')} />

      {NEEDS_AREAS.map((a) => (
        <SectionCard key={a.key} title={a.label} hint={a.hint} defaultOpen={false}>
          <TextArea
            value={form.areas[a.key]}
            onChange={(v) => updateArea(a.key, v)}
            placeholder="의견 및 판단근거: 체크리스트 요약이 아닌, 왜 그렇게 평가했는지 구체적 사실로 서술"
            minRows={3}
          />
        </SectionCard>
      ))}

      <SectionCard title="종합의견" hint="8개 영역을 묶는 전체 종합 의견">
        <TextArea value={form.overview} onChange={(v) => setForm((f) => ({ ...f, overview: v }))} minRows={3} placeholder="8개 영역을 종합한 전체 의견 작성" />
      </SectionCard>

      <SaveBar onSave={save} savedAt={form.savedAt} saving={saving} />
      <OutputBox text={output} label={`욕구조사기록지 — ${form.date} (${form.type})`} />
    </div>
  );
}

function buildNeedsText(recipient, form) {
  const lines = [];
  lines.push('[욕구조사기록지]');
  lines.push(`수급자: ${recipient.name}`);
  lines.push(`작성일: ${form.date} / 조사유형: ${form.type}`);
  lines.push('');
  NEEDS_AREAS.forEach((a) => {
    lines.push(`■ ${a.label}`);
    lines.push(`의견 및 판단근거: ${form.areas[a.key]?.trim() || '특이사항 없어 기존과 동일하게 유지함'}`);
    lines.push('');
  });
  lines.push('■ 종합의견');
  lines.push(form.overview?.trim() || '8개 영역 종합 결과 특이 변화사항 없음');
  return lines.join('\n');
}

const naNewBtnStyle = {
  padding: '7px 12px',
  borderRadius: 7,
  border: '1px solid #1F4D3D',
  background: '#FFFFFF',
  color: '#1F4D3D',
  fontSize: 12.5,
  fontWeight: 700,
  cursor: 'pointer',
};

const naTypeBtnStyle = {
  flex: 1,
  padding: '7px 0',
  borderRadius: 7,
  border: '1px solid #E0DACE',
  background: '#FAF8F4',
  color: '#8A8275',
  fontSize: 12.5,
  fontWeight: 600,
  cursor: 'pointer',
};

const naTypeBtnActive = {
  background: '#1F4D3D',
  color: '#FFF',
  borderColor: '#1F4D3D',
};


// ===== ResultEval.jsx =====


const FUTURE_PLAN_OPTIONS = ['계획유지', '욕구조사 재실시 및 계획서 재작성', '사례회의 실시', '기타'];

function reEmptyGoal() {
  return { id: 'g_' + Date.now() + Math.random().toString(36).slice(2, 5), serviceType: '', goal: '', detail: '', rate: '', provided: '제공', result: '' };
}

function reEmptyAreas() {
  return NINE_AREAS.reduce((acc, a) => {
    acc[a.key] = { status: '유지', reason: '' };
    return acc;
  }, {});
}

function reEmptyForm() {
  return {
    date: todayStr(),
    period: '',
    goals: [reEmptyGoal()],
    summary: '',
    areas: reEmptyAreas(),
    needsChanged: '미변화',
    futurePlan: ['계획유지'],
    futurePlanNote: '',
    savedAt: null,
  };
}

function ResultEval({ recipient }) {
  const [form, setForm] = useState(reEmptyForm());
  const [refText, setRefText] = useState('');
  const [saving, setSaving] = useState(false);
  const [records, setRecords] = useState([]);
  const [activeKey, setActiveKey] = useState(null);

  const loadRecords = useCallback(async () => {
    const keys = await safeList(`resulteval:${recipient.id}:`, true);
    setRecords(keys.sort().reverse());
  }, [recipient.id]);

  useEffect(() => {
    loadRecords();
    setForm(reEmptyForm());
    setActiveKey(null);
  }, [recipient.id, loadRecords]);

  const loadRecord = async (key) => {
    const raw = await safeGet(key, true);
    if (raw) {
      try {
        const parsed = JSON.parse(raw);
        setForm({
          ...reEmptyForm(),
          ...parsed,
          areas: { ...reEmptyAreas(), ...(parsed.areas || {}) },
          goals: parsed.goals?.length ? parsed.goals : [reEmptyGoal()],
        });
        setActiveKey(key);
      } catch (e) {}
    }
  };

  const startNew = () => {
    setForm(reEmptyForm());
    setActiveKey(null);
  };

  const save = async () => {
    setSaving(true);
    const savedAt = new Date().toLocaleString('ko-KR');
    const key = activeKey || `resulteval:${recipient.id}:${form.date}_${Date.now()}`;
    const toSave = { ...form, savedAt };
    await safeSet(key, JSON.stringify(toSave), true);
    setForm(toSave);
    setActiveKey(key);
    await loadRecords();
    setSaving(false);
  };

  const updateGoal = (id, field, value) => {
    setForm((f) => ({ ...f, goals: f.goals.map((g) => (g.id === id ? { ...g, [field]: value } : g)) }));
  };
  const addGoal = () => setForm((f) => ({ ...f, goals: [...f.goals, reEmptyGoal()] }));
  const removeGoal = (id) => setForm((f) => ({ ...f, goals: f.goals.filter((g) => g.id !== id) }));

  const updateArea = (key, field, value) => {
    setForm((f) => ({ ...f, areas: { ...f.areas, [key]: { ...f.areas[key], [field]: value } } }));
  };

  const toggleFuturePlan = (opt) => {
    setForm((f) => {
      const has = f.futurePlan.includes(opt);
      return { ...f, futurePlan: has ? f.futurePlan.filter((o) => o !== opt) : [...f.futurePlan, opt] };
    });
  };

  const output = useMemo(() => buildResultText(recipient, form), [recipient, form]);

  return (
    <div>
      <SectionCard title="평가 기본정보" defaultOpen={true}>
        <div style={{ display: 'flex', gap: 8, marginBottom: 8 }}>
          <button onClick={startNew} style={reNewBtnStyle}>+ 새 평가 작성</button>
        </div>
        {records.length > 0 && (
          <select
            value={activeKey || ''}
            onChange={(e) => (e.target.value ? loadRecord(e.target.value) : startNew())}
            style={{ width: '100%', padding: '9px 10px', borderRadius: 8, border: '1px solid #DDD5C7', fontSize: 13, background: '#FFFDF9', marginBottom: 8 }}
          >
            <option value="">(작성 중 — 새 기록)</option>
            {records.map((k) => (
              <option key={k} value={k}>{k.split(':')[2]?.split('_')[0]}</option>
            ))}
          </select>
        )}
        <div style={{ display: 'flex', gap: 8 }}>
          <TextField value={form.date} onChange={(v) => setForm((f) => ({ ...f, date: v }))} placeholder="평가일 (YYYY-MM-DD)" style={{ flex: 1 }} />
          <TextField value={form.period} onChange={(v) => setForm((f) => ({ ...f, period: v }))} placeholder="평가대상 서비스기간" style={{ flex: 1 }} />
        </div>
      </SectionCard>

      <PasteImportBox onImport={setRefText} />
      <ReferencePanel text={refText} onClear={() => setRefText('')} />

      <SectionCard title="① 급여종류별 세부목표 달성률 및 총평" hint="급여종류는 자유 입력 (예: 방문요양, 방문목욕 등)">
        {form.goals.map((g, idx) => (
          <div key={g.id} style={reGoalBoxStyle}>
            <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
              <span style={{ fontSize: 12.5, fontWeight: 700, color: '#8A7A4A' }}>급여 {idx + 1}</span>
              {form.goals.length > 1 && (
                <button onClick={() => removeGoal(g.id)} style={reRemoveBtnStyle}><X size={13} /></button>
              )}
            </div>
            <TextField value={g.serviceType} onChange={(v) => updateGoal(g.id, 'serviceType', v)} placeholder="급여종류 (예: 방문요양)" />
            <TextArea value={g.goal} onChange={(v) => updateGoal(g.id, 'goal', v)} placeholder="세부목표 (급여제공계획서의 장기요양세부목표 참고)" minRows={2} />
            <TextArea value={g.detail} onChange={(v) => updateGoal(g.id, 'detail', v)} placeholder="세부내용/제공결과 (서비스 제공에 따른 수급자 상태 변화)" minRows={2} />
            <div style={{ display: 'flex', gap: 8 }}>
              <TextField value={g.rate} onChange={(v) => updateGoal(g.id, 'rate', v)} placeholder="달성률 (%)" style={{ flex: 1 }} />
              <select
                value={g.provided}
                onChange={(e) => updateGoal(g.id, 'provided', e.target.value)}
                style={{ flex: 1, padding: '9px 10px', borderRadius: 8, border: '1px solid #DDD5C7', fontSize: 13, background: '#FFFDF9' }}
              >
                <option value="제공">제공</option>
                <option value="부분제공">부분제공</option>
                <option value="미제공">미제공</option>
              </select>
            </div>
            <TextArea value={g.result} onChange={(v) => updateGoal(g.id, 'result', v)} placeholder="평가 (담당자/기관 평가 및 계획 재수립 시 반영방향)" minRows={2} />
          </div>
        ))}
        <button onClick={addGoal} style={reAddGoalBtn}><Plus size={14} /> 급여 추가</button>
        <div style={{ marginTop: 4 }}>
          <div style={{ fontSize: 12.5, fontWeight: 700, marginBottom: 6 }}>총평</div>
          <TextArea value={form.summary} onChange={(v) => setForm((f) => ({ ...f, summary: v }))} placeholder="급여제공 전반에 대한 총평" minRows={3} />
        </div>
      </SectionCard>

      <SectionCard title="② 심신상태변화 (9개 영역)" hint="업무수행일지와 동일한 형식">
        {NINE_AREAS.map((a) => (
          <div key={a.key} style={reAreaBoxStyle}>
            <div style={reAreaTitleRow}>
              <div>
                <div style={{ fontSize: 13.5, fontWeight: 700 }}>{a.label}</div>
                <div style={{ fontSize: 11, color: '#A39B8B' }}>{a.hint}</div>
              </div>
              <StatusPicker value={form.areas[a.key]?.status} onChange={(v) => updateArea(a.key, 'status', v)} options={STATUS_OPTIONS} />
            </div>
            {form.areas[a.key]?.status !== '유지' && (
              <TextArea
                value={form.areas[a.key]?.reason}
                onChange={(v) => updateArea(a.key, 'reason', v)}
                placeholder="판단근거: 구체적 사실 기재"
                minRows={2}
              />
            )}
          </div>
        ))}
      </SectionCard>

      <SectionCard title="③ 종합의견 및 향후계획">
        <div style={{ fontSize: 12.5, fontWeight: 700, marginBottom: 4 }}>욕구/기능상태 변화여부</div>
        <div style={{ display: 'flex', gap: 6 }}>
          {['미변화', '변화있음'].map((opt) => (
            <button
              key={opt}
              onClick={() => setForm((f) => ({ ...f, needsChanged: opt }))}
              style={{ ...reTypeBtnStyle, ...(form.needsChanged === opt ? reTypeBtnActive : {}) }}
            >
              {opt}
            </button>
          ))}
        </div>

        <div style={{ fontSize: 12.5, fontWeight: 700, marginTop: 10, marginBottom: 4 }}>해당 향후계획 (복수선택 가능)</div>
        <div style={{ display: 'flex', flexWrap: 'wrap', gap: 6 }}>
          {FUTURE_PLAN_OPTIONS.map((opt) => (
            <button
              key={opt}
              onClick={() => toggleFuturePlan(opt)}
              style={{ ...reChipStyle, ...(form.futurePlan.includes(opt) ? reChipActive : {}) }}
            >
              {opt}
            </button>
          ))}
        </div>
        <TextArea
          value={form.futurePlanNote}
          onChange={(v) => setForm((f) => ({ ...f, futurePlanNote: v }))}
          placeholder="향후계획 관련 부연설명 (선택)"
          minRows={2}
        />
      </SectionCard>

      <SaveBar onSave={save} savedAt={form.savedAt} saving={saving} />
      <OutputBox text={output} label={`급여제공결과평가지 — ${form.date}`} />
    </div>
  );
}

function buildResultText(recipient, form) {
  const lines = [];
  lines.push('[급여제공결과평가지]');
  lines.push(`수급자: ${recipient.name}`);
  lines.push(`평가일: ${form.date}${form.period ? ` / 평가대상기간: ${form.period}` : ''}`);
  lines.push('');
  lines.push('■ ① 급여종류별 세부목표 달성률 및 제공여부');
  form.goals.forEach((g, idx) => {
    lines.push(`[급여 ${idx + 1}] ${g.serviceType?.trim() || '(급여종류 미기재)'}`);
    lines.push(`- 세부목표: ${g.goal?.trim() || '미기재'}`);
    lines.push(`- 세부내용/제공결과: ${g.detail?.trim() || '미기재'}`);
    lines.push(`- 달성률: ${g.rate?.trim() || '미기재'}% / 제공여부: ${g.provided}`);
    lines.push(`- 평가: ${g.result?.trim() || '미기재'}`);
    lines.push('');
  });
  lines.push('총평');
  lines.push(form.summary?.trim() || '급여제공계획에 따라 서비스가 적절히 제공됨');
  lines.push('');
  lines.push('■ ② 심신상태변화 (9개 영역)');
  NINE_AREAS.forEach((a) => {
    const area = form.areas[a.key] || {};
    const status = area.status || '유지';
    lines.push(`- ${a.label}: ${status}${status !== '유지' && area.reason?.trim() ? ` (${area.reason.trim()})` : ''}`);
  });
  lines.push('');
  lines.push('■ ③ 종합의견 및 향후계획');
  lines.push(`욕구/기능상태 변화여부: ${form.needsChanged}`);
  lines.push(`향후계획: ${form.futurePlan.join(', ') || '계획유지'}`);
  if (form.futurePlanNote?.trim()) lines.push(form.futurePlanNote.trim());
  return lines.join('\n');
}

const reNewBtnStyle = {
  padding: '7px 12px',
  borderRadius: 7,
  border: '1px solid #1F4D3D',
  background: '#FFFFFF',
  color: '#1F4D3D',
  fontSize: 12.5,
  fontWeight: 700,
  cursor: 'pointer',
};

const reGoalBoxStyle = {
  border: '1px solid #EFE9DC',
  borderRadius: 8,
  padding: 10,
  display: 'flex',
  flexDirection: 'column',
  gap: 8,
  background: '#FCFAF6',
};

const reRemoveBtnStyle = {
  border: 'none',
  background: 'transparent',
  color: '#A3401F',
  cursor: 'pointer',
  padding: 2,
};

const reAddGoalBtn = {
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center',
  gap: 6,
  padding: '8px 0',
  borderRadius: 8,
  border: '1px dashed #D2C8B4',
  background: 'transparent',
  color: '#8A7A4A',
  fontSize: 12.5,
  fontWeight: 600,
  cursor: 'pointer',
};

const reAreaBoxStyle = {
  border: '1px solid #EFE9DC',
  borderRadius: 8,
  padding: 10,
  display: 'flex',
  flexDirection: 'column',
  gap: 8,
  background: '#FCFAF6',
};

const reAreaTitleRow = {
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'space-between',
  gap: 10,
};

const reTypeBtnStyle = {
  flex: 1,
  padding: '7px 0',
  borderRadius: 7,
  border: '1px solid #E0DACE',
  background: '#FAF8F4',
  color: '#8A8275',
  fontSize: 12.5,
  fontWeight: 600,
  cursor: 'pointer',
};

const reTypeBtnActive = {
  background: '#1F4D3D',
  color: '#FFF',
  borderColor: '#1F4D3D',
};

const reChipStyle = {
  padding: '6px 12px',
  borderRadius: 16,
  border: '1px solid #E0DACE',
  background: '#FAF8F4',
  color: '#8A8275',
  fontSize: 12,
  fontWeight: 600,
  cursor: 'pointer',
};

const reChipActive = {
  background: '#1F4D3D',
  color: '#FFF',
  borderColor: '#1F4D3D',
};


// ===== RewriteCheck.jsx =====


const CONTRACT_EVENTS = [
  { key: 'none', label: '변동 없음' },
  { key: 'terminated_renewed', label: '계약해지 후 재계약' },
];

const CERT_TYPES = [
  { key: 'new', label: '신규' },
  { key: 'renewal', label: '갱신 (등급 변동 무관)' },
  { key: 'grade_change', label: '등급변경' },
  { key: 'auto_extend', label: '자동연장 (재방문조사·등급판정 없는 행정적 연장)' },
  { key: 'none', label: '해당없음 / 변동없음' },
];

function RewriteCheck({ recipient }) {
  const [contractEvent, setContractEvent] = useState('none');
  const [certType, setCertType] = useState('none');
  const [certExpiry, setCertExpiry] = useState('');
  const [functionChanged, setFunctionChanged] = useState(false);
  const [serviceTimeChanged, setServiceTimeChanged] = useState(false);
  const [lastNeedsDate, setLastNeedsDate] = useState('');
  const [lastPlanDate, setLastPlanDate] = useState('');
  const [note, setNote] = useState('');

  const result = useMemo(
    () => evaluate({ contractEvent, certType, functionChanged, serviceTimeChanged }),
    [contractEvent, certType, functionChanged, serviceTimeChanged]
  );

  return (
    <div>
      <SectionCard title="계약 관련" defaultOpen={true}>
        <div style={{ display: 'flex', flexDirection: 'column', gap: 6 }}>
          {CONTRACT_EVENTS.map((opt) => (
            <Radio key={opt.key} checked={contractEvent === opt.key} onClick={() => setContractEvent(opt.key)} label={opt.label} />
          ))}
        </div>
      </SectionCard>

      <SectionCard title="인정서 발급유형" defaultOpen={true}>
        <div style={{ display: 'flex', flexDirection: 'column', gap: 6 }}>
          {CERT_TYPES.map((opt) => (
            <Radio key={opt.key} checked={certType === opt.key} onClick={() => setCertType(opt.key)} label={opt.label} />
          ))}
        </div>
        <TextField value={certExpiry} onChange={setCertExpiry} placeholder="인정유효기간 만료일자 (YYYY-MM-DD)" />
      </SectionCard>

      <SectionCard title="기능상태·욕구 / 서비스 변경" defaultOpen={true}>
        <Checkbox checked={functionChanged} onClick={() => setFunctionChanged((v) => !v)} label="기능상태·욕구 변화가 확인됨" />
        <Checkbox checked={serviceTimeChanged} onClick={() => setServiceTimeChanged((v) => !v)} label="서비스시간/급여종류만 변경됨 (다른 변동 없음)" />
      </SectionCard>

      <SectionCard title="최근 작성일 (참고용)" defaultOpen={false}>
        <div style={{ display: 'flex', gap: 8 }}>
          <TextField value={lastNeedsDate} onChange={setLastNeedsDate} placeholder="최근 욕구조사 작성일" style={{ flex: 1 }} />
          <TextField value={lastPlanDate} onChange={setLastPlanDate} placeholder="최근 계획서 작성일" style={{ flex: 1 }} />
        </div>
        <TextArea value={note} onChange={setNote} placeholder="추가 메모 (선택)" minRows={2} />
      </SectionCard>

      <ResultCard result={result} recipient={recipient} certExpiry={certExpiry} lastNeedsDate={lastNeedsDate} lastPlanDate={lastPlanDate} note={note} />
    </div>
  );
}

function evaluate({ contractEvent, certType, functionChanged, serviceTimeChanged }) {
  // 우선순위: 계약해지 후 재계약 / 인정서 갱신·등급변경 / 기능상태 변화 -> 욕구조사+계획서 모두 재작성
  const fullReasons = [];
  if (contractEvent === 'terminated_renewed') fullReasons.push('계약해지 후 재계약이 발생함');
  if (certType === 'renewal') fullReasons.push('인정서가 갱신됨(등급 변동 여부와 무관하게 재방문조사·등급판정을 거친 갱신임)');
  if (certType === 'grade_change') fullReasons.push('장기요양등급이 변경됨');
  if (functionChanged) fullReasons.push('기능상태·욕구 변화가 확인됨');

  if (fullReasons.length > 0) {
    return {
      level: 'full',
      title: '욕구조사 + 급여제공계획서 모두 재작성 필요',
      reasons: fullReasons,
    };
  }

  if (certType === 'auto_extend') {
    return {
      level: 'none',
      title: '재작성 불필요 — 만료일자만 기록 갱신',
      reasons: ['인정기간이 재방문조사·등급판정 없는 행정적 자동연장으로 처리됨'],
    };
  }

  if (serviceTimeChanged) {
    return {
      level: 'plan_only',
      title: '급여제공계획서만 재작성 권장',
      reasons: ['서비스시간 또는 급여종류만 변경되고 그 외 변동사항은 없음'],
    };
  }

  return {
    level: 'none',
    title: '재작성 불필요',
    reasons: ['계약·인정서·기능상태·서비스 구성에 재작성을 요하는 변동사항이 확인되지 않음'],
  };
}

function ResultCard({ result, recipient, certExpiry, lastNeedsDate, lastPlanDate, note }) {
  const cfg = {
    full: { color: '#A3401F', bg: '#FBEDEA', Icon: AlertTriangle },
    plan_only: { color: '#8A6D1A', bg: '#FBF3E0', Icon: RefreshCcw },
    none: { color: '#1F4D3D', bg: '#EAF1ED', Icon: CheckCircle2 },
  }[result.level];
  const { Icon } = cfg;

  return (
    <div style={{ ...resultBox, background: cfg.bg, borderColor: cfg.color }}>
      <div style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
        <Icon size={20} color={cfg.color} />
        <div style={{ fontSize: 15, fontWeight: 800, color: cfg.color }}>{result.title}</div>
      </div>
      <ul style={{ margin: '10px 0 0', paddingLeft: 18, display: 'flex', flexDirection: 'column', gap: 4 }}>
        {result.reasons.map((r, i) => (
          <li key={i} style={{ fontSize: 13, color: '#4A4435', lineHeight: 1.6 }}>{r}</li>
        ))}
      </ul>
      {certExpiry && (
        <div style={{ marginTop: 10, fontSize: 12.5, color: '#8A8275' }}>
          인정유효기간 만료일자: <b style={{ color: '#2E2A24' }}>{certExpiry}</b>
        </div>
      )}
      {(lastNeedsDate || lastPlanDate) && (
        <div style={{ marginTop: 4, fontSize: 12.5, color: '#8A8275' }}>
          {lastNeedsDate && <>최근 욕구조사: {lastNeedsDate}　</>}
          {lastPlanDate && <>최근 계획서: {lastPlanDate}</>}
        </div>
      )}
      <div style={{ marginTop: 12, paddingTop: 12, borderTop: `1px solid ${cfg.color}33` }}>
        <div style={{ fontSize: 11.5, fontWeight: 700, color: cfg.color, marginBottom: 6 }}>복사용 텍스트</div>
        <pre style={{ margin: 0, fontSize: 12.5, lineHeight: 1.7, whiteSpace: 'pre-wrap', color: '#3A3528' }}>
{`[서류 재작성 판단]
수급자: ${recipient.name}
판정: ${result.title}
사유: ${result.reasons.join(' / ')}${certExpiry ? `\n인정유효기간 만료일자: ${certExpiry}` : ''}${note?.trim() ? `\n메모: ${note.trim()}` : ''}`}
        </pre>
      </div>
    </div>
  );
}

function Radio({ checked, onClick, label }) {
  return (
    <div onClick={onClick} style={radioRow}>
      <div style={{ ...radioCircle, ...(checked ? radioCircleActive : {}) }}>
        {checked && <div style={radioDot} />}
      </div>
      <span style={{ fontSize: 13, color: checked ? '#2E2A24' : '#6B6354', fontWeight: checked ? 600 : 400 }}>{label}</span>
    </div>
  );
}

function Checkbox({ checked, onClick, label }) {
  return (
    <div onClick={onClick} style={radioRow}>
      <div style={{ ...checkBox, ...(checked ? checkBoxActive : {}) }}>
        {checked && <span style={{ color: '#FFF', fontSize: 11, fontWeight: 900 }}>✓</span>}
      </div>
      <span style={{ fontSize: 13, color: checked ? '#2E2A24' : '#6B6354', fontWeight: checked ? 600 : 400 }}>{label}</span>
    </div>
  );
}

const resultBox = {
  border: '1.5px solid',
  borderRadius: 12,
  padding: 16,
  marginTop: 14,
};

const radioRow = {
  display: 'flex',
  alignItems: 'center',
  gap: 9,
  cursor: 'pointer',
  padding: '5px 2px',
};

const radioCircle = {
  width: 17,
  height: 17,
  borderRadius: '50%',
  border: '1.5px solid #C9C0AE',
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center',
  flexShrink: 0,
};

const radioCircleActive = { borderColor: '#1F4D3D' };

const radioDot = {
  width: 9,
  height: 9,
  borderRadius: '50%',
  background: '#1F4D3D',
};

const checkBox = {
  width: 17,
  height: 17,
  borderRadius: 5,
  border: '1.5px solid #C9C0AE',
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center',
  flexShrink: 0,
};

const checkBoxActive = { background: '#1F4D3D', borderColor: '#1F4D3D' };


// ===== App.jsx =====


const TABS = [
  { key: 'worklog', label: '업무수행일지', icon: FileText },
  { key: 'needs', label: '욕구조사기록지', icon: ClipboardList },
  { key: 'result', label: '급여제공결과평가지', icon: BarChart3 },
  { key: 'rewrite', label: '재작성 판단', icon: GitCompare },
];

function App() {
  const [recipients, setRecipients] = useState([]);
  const [loading, setLoading] = useState(true);
  const [selectedId, setSelectedId] = useState(null);
  const [activeTab, setActiveTab] = useState('worklog');
  const [showAddModal, setShowAddModal] = useState(false);
  const [newName, setNewName] = useState('');
  const [search, setSearch] = useState('');
  const [deleteTarget, setDeleteTarget] = useState(null);

  const loadRecipients = useCallback(async () => {
    setLoading(true);
    const keys = await safeList('recipient:', true);
    const list = [];
    for (const k of keys) {
      const val = await safeGet(k, true);
      if (val) {
        try {
          list.push(JSON.parse(val));
        } catch (e) {}
      }
    }
    list.sort((a, b) => (a.name || '').localeCompare(b.name || '', 'ko'));
    setRecipients(list);
    setLoading(false);
  }, []);

  useEffect(() => {
    loadRecipients();
  }, [loadRecipients]);

  const addRecipient = async () => {
    const name = newName.trim();
    if (!name) return;
    const id = 'r_' + Date.now() + '_' + Math.random().toString(36).slice(2, 7);
    const recipient = { id, name, createdAt: todayStr() };
    await safeSet(`recipient:${id}`, JSON.stringify(recipient), true);
    setNewName('');
    setShowAddModal(false);
    await loadRecipients();
    setSelectedId(id);
  };

  const requestRemove = (id, name, e) => {
    e.stopPropagation();
    setDeleteTarget({ id, name });
  };

  const confirmRemove = async () => {
    if (!deleteTarget) return;
    const id = deleteTarget.id;
    setDeleteTarget(null);
    await safeDelete(`recipient:${id}`, true);
    await loadRecipients();
    if (selectedId === id) setSelectedId(null);
  };

  const selected = recipients.find((r) => r.id === selectedId);
  const filtered = recipients.filter((r) => r.name.includes(search.trim()));

  if (!selected) {
    return (
      <div style={styles.shell}>
        <div style={styles.header}>
          <div style={styles.headerTitle}>
            <Users size={22} strokeWidth={2.2} />
            <span>방문요양 업무지원</span>
          </div>
          <div style={styles.headerSub}>수급자를 선택하면 서류 작성을 시작합니다</div>
        </div>

        <div style={styles.searchRow}>
          <input
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            placeholder="수급자 이름 검색"
            style={styles.searchInput}
          />
          <button style={styles.addBtn} onClick={() => setShowAddModal(true)}>
            <Plus size={16} /> 수급자 추가
          </button>
        </div>

        {loading ? (
          <div style={styles.emptyState}>
            <Loader2 size={20} className="spin" style={{ animation: 'spin 1s linear infinite' }} />
            <span>불러오는 중...</span>
          </div>
        ) : filtered.length === 0 ? (
          <div style={styles.emptyState}>
            <Users size={28} color="#B8AFA0" />
            <div style={{ color: '#8A8275', fontSize: 14 }}>
              {recipients.length === 0 ? '등록된 수급자가 없습니다. 먼저 수급자를 추가해 주세요.' : '검색 결과가 없습니다.'}
            </div>
          </div>
        ) : (
          <div style={styles.list}>
            {filtered.map((r) => (
              <div key={r.id} style={styles.listItem} onClick={() => setSelectedId(r.id)}>
                <div style={styles.avatar}>{r.name.slice(0, 1)}</div>
                <div style={{ flex: 1 }}>
                  <div style={styles.listName}>{r.name}</div>
                  <div style={styles.listMeta}>등록일 {r.createdAt}</div>
                </div>
                <button style={styles.delBtn} onClick={(e) => requestRemove(r.id, r.name, e)}>삭제</button>
                <ChevronRight size={18} color="#B8AFA0" />
              </div>
            ))}
          </div>
        )}

        {showAddModal && (
          <div style={styles.modalOverlay} onClick={() => setShowAddModal(false)}>
            <div style={styles.modal} onClick={(e) => e.stopPropagation()}>
              <div style={styles.modalTitle}>수급자 추가</div>
              <input
                autoFocus
                value={newName}
                onChange={(e) => setNewName(e.target.value)}
                onKeyDown={(e) => e.key === 'Enter' && addRecipient()}
                placeholder="수급자 성명"
                style={styles.modalInput}
              />
              <div style={styles.modalActions}>
                <button style={styles.modalCancel} onClick={() => setShowAddModal(false)}>취소</button>
                <button style={styles.modalConfirm} onClick={addRecipient}>추가</button>
              </div>
            </div>
          </div>
        )}

        {deleteTarget && (
          <div style={styles.modalOverlay} onClick={() => setDeleteTarget(null)}>
            <div style={styles.modal} onClick={(e) => e.stopPropagation()}>
              <div style={styles.modalTitle}>수급자 삭제</div>
              <div style={{ fontSize: 13, color: '#6B6354', lineHeight: 1.6, marginBottom: 4 }}>
                <b style={{ color: '#2E2A24' }}>{deleteTarget.name}</b>님을 목록에서 삭제할까요?
                <br />
                작성된 서류 기록은 별도로 보관되며 목록 진입만 제거됩니다.
              </div>
              <div style={styles.modalActions}>
                <button style={styles.modalCancel} onClick={() => setDeleteTarget(null)}>취소</button>
                <button style={styles.modalDanger} onClick={confirmRemove}>삭제</button>
              </div>
            </div>
          </div>
        )}
        <GlobalStyle />
      </div>
    );
  }

  return (
    <div style={styles.shell}>
      <div style={styles.topBar}>
        <button style={styles.backBtn} onClick={() => setSelectedId(null)}>
          ← 목록
        </button>
        <div style={styles.recipientName}>{selected.name}</div>
        <div style={{ width: 56 }} />
      </div>

      <div style={styles.tabBar}>
        {TABS.map((t) => {
          const Icon = t.icon;
          const active = activeTab === t.key;
          return (
            <button
              key={t.key}
              onClick={() => setActiveTab(t.key)}
              style={{ ...styles.tabBtn, ...(active ? styles.tabBtnActive : {}) }}
            >
              <Icon size={15} strokeWidth={2.2} />
              {t.label}
            </button>
          );
        })}
      </div>

      <div style={styles.content}>
        {activeTab === 'worklog' && <WorkLog recipient={selected} />}
        {activeTab === 'needs' && <NeedsAssessment recipient={selected} />}
        {activeTab === 'result' && <ResultEval recipient={selected} />}
        {activeTab === 'rewrite' && <RewriteCheck recipient={selected} />}
      </div>
      <GlobalStyle />
    </div>
  );
}

function GlobalStyle() {
  return (
    <style>{`
      @keyframes spin { from { transform: rotate(0deg);} to { transform: rotate(360deg);} }
      * { box-sizing: border-box; }
      input, textarea, select, button { font-family: inherit; }
      ::placeholder { color: #B8AFA0; }
    `}</style>
  );
}

const styles = {
  shell: {
    fontFamily: "'Pretendard', -apple-system, 'Apple SD Gothic Neo', 'Noto Sans KR', sans-serif",
    background: '#FAF8F4',
    minHeight: '100%',
    color: '#2E2A24',
    maxWidth: 720,
    margin: '0 auto',
  },
  header: {
    padding: '28px 20px 18px',
    borderBottom: '1px solid #E8E2D6',
  },
  headerTitle: {
    display: 'flex',
    alignItems: 'center',
    gap: 8,
    fontSize: 19,
    fontWeight: 700,
    color: '#1F4D3D',
    letterSpacing: '-0.02em',
  },
  headerSub: {
    marginTop: 4,
    fontSize: 13,
    color: '#8A8275',
  },
  searchRow: {
    display: 'flex',
    gap: 8,
    padding: '14px 20px',
  },
  searchInput: {
    flex: 1,
    padding: '10px 12px',
    borderRadius: 8,
    border: '1px solid #DDD5C7',
    fontSize: 14,
    outline: 'none',
    background: '#FFFFFF',
  },
  addBtn: {
    display: 'flex',
    alignItems: 'center',
    gap: 6,
    padding: '10px 14px',
    borderRadius: 8,
    border: 'none',
    background: '#1F4D3D',
    color: '#FFF',
    fontSize: 13,
    fontWeight: 600,
    cursor: 'pointer',
    whiteSpace: 'nowrap',
  },
  emptyState: {
    display: 'flex',
    flexDirection: 'column',
    alignItems: 'center',
    justifyContent: 'center',
    gap: 10,
    padding: '60px 20px',
  },
  list: {
    padding: '0 20px 20px',
    display: 'flex',
    flexDirection: 'column',
    gap: 8,
  },
  listItem: {
    display: 'flex',
    alignItems: 'center',
    gap: 12,
    padding: '12px 14px',
    background: '#FFFFFF',
    border: '1px solid #EDE7DA',
    borderRadius: 10,
    cursor: 'pointer',
  },
  avatar: {
    width: 36,
    height: 36,
    borderRadius: '50%',
    background: '#EAF1ED',
    color: '#1F4D3D',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    fontWeight: 700,
    fontSize: 14,
    flexShrink: 0,
  },
  listName: { fontSize: 14.5, fontWeight: 600 },
  listMeta: { fontSize: 12, color: '#A39B8B', marginTop: 2 },
  delBtn: {
    fontSize: 11.5,
    color: '#B0473F',
    background: 'transparent',
    border: '1px solid #EAD3D0',
    borderRadius: 6,
    padding: '4px 8px',
    cursor: 'pointer',
  },
  modalOverlay: {
    position: 'fixed',
    inset: 0,
    background: 'rgba(30,26,20,0.4)',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    zIndex: 50,
  },
  modal: {
    background: '#FFF',
    borderRadius: 12,
    padding: 20,
    width: 300,
  },
  modalTitle: { fontSize: 15, fontWeight: 700, marginBottom: 12 },
  modalInput: {
    width: '100%',
    padding: '10px 12px',
    borderRadius: 8,
    border: '1px solid #DDD5C7',
    fontSize: 14,
    outline: 'none',
  },
  modalActions: { display: 'flex', gap: 8, marginTop: 14, justifyContent: 'flex-end' },
  modalCancel: {
    padding: '8px 14px',
    borderRadius: 7,
    border: '1px solid #DDD5C7',
    background: '#FFF',
    fontSize: 13,
    cursor: 'pointer',
  },
  modalConfirm: {
    padding: '8px 14px',
    borderRadius: 7,
    border: 'none',
    background: '#1F4D3D',
    color: '#FFF',
    fontSize: 13,
    fontWeight: 600,
    cursor: 'pointer',
  },
  modalDanger: {
    padding: '8px 14px',
    borderRadius: 7,
    border: 'none',
    background: '#B0473F',
    color: '#FFF',
    fontSize: 13,
    fontWeight: 600,
    cursor: 'pointer',
  },
  topBar: {
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: '16px 16px 10px',
  },
  backBtn: {
    background: 'transparent',
    border: 'none',
    color: '#1F4D3D',
    fontSize: 13.5,
    fontWeight: 600,
    cursor: 'pointer',
    padding: '6px 4px',
  },
  recipientName: { fontSize: 16, fontWeight: 700 },
  tabBar: {
    display: 'flex',
    gap: 4,
    padding: '0 12px',
    borderBottom: '1px solid #E8E2D6',
    overflowX: 'auto',
  },
  tabBtn: {
    display: 'flex',
    alignItems: 'center',
    gap: 5,
    padding: '10px 12px',
    border: 'none',
    background: 'transparent',
    fontSize: 12.5,
    fontWeight: 600,
    color: '#A39B8B',
    cursor: 'pointer',
    borderBottom: '2px solid transparent',
    whiteSpace: 'nowrap',
  },
  tabBtnActive: {
    color: '#1F4D3D',
    borderBottom: '2px solid #1F4D3D',
  },
  content: {
    padding: '16px 16px 40px',
  },
};


export default App;
