import React, { useState, useEffect, useRef } from 'react';
import { 
  X, ChevronRight, Settings, Save, Plus, Trash2, Loader2, ArrowUpRight,
  MapPin, Briefcase, GraduationCap, PenTool, Code, Mail, Instagram, 
  ChevronLeft, BookOpen, Layers, Image as ImageIcon, Link as LinkIcon,
  Heart, MessageCircle, Send, Bookmark, MoreHorizontal, FileText, Download,
  ExternalLink, Eye
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged 
} from 'firebase/auth';
import { 
  getFirestore, doc, setDoc, onSnapshot 
} from 'firebase/firestore';

/**
 * ==========================================
 * 🛡️ Firebase 配置與初始化
 * ==========================================
 */
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'design-portfolio-final';

// 初始預設資料
const INITIAL_DATA = {
  profile: {
    name: "王郁平",
    nameEn: "YuPing Wang",
    title: "Graphic Designer / Visual Artist",
    location: "Taipei, Taiwan",
    email: "phinainaina@gmail.com",
    instagram: "https://instagram.com",
    intro: "「從品牌的靈魂雕琢到大型商業專刊的精密排版，我致力於透過視覺語言與 AI 協作，為客戶創造最具前瞻性的商業價值。」",
  },
  softwareIcons: {
    photoshop: "skill_image/skill_ps.jpg",
    illustrator: "skill_image/skill_ai.jpg",
    indesign: "skill_image/skill_id.jpg",
    premiere: "skill_image/skill_pr.jpg",
    firefly: "skill_image/skill_fi.jpg",
    gemini: "skill_image/skill_gimini.jpg",
    suno: "skill_image/skill_suno.jpg",
    cursor: "skill_image/skill_Cursor.jpg"
  },
  experience: [
    { id: 'exp1', period: "2024.07 – 至今", role: "平面視覺設計師", company: "來電司康 ENSCON", desc: "專注於品牌核心視覺發展與多維度設計執行。" },
    { id: 'exp2', period: "2022.02 – 2024.06", role: "自由設計師 (SOHO)", company: "獨立承接 / Freelance", desc: "負責跨產業品牌設計，完成全流程視覺規劃。" },
    { id: 'exp3', period: "2020.12 – 2022.02", role: "電商品牌設計專員", company: "十方力有限公司", desc: "主導官網視覺與包裝設計，串聯行銷與營運需求。" },
    { id: 'exp4', period: "2017.09 – 2019.06", role: "美工宣傳專員", company: "太平洋崇光百貨 (SOGO)", desc: "執行天母館大型檔期專刊排版。" }
  ],
  education: [
    { id: 'edu1', period: "2018 - 2022", role: "視覺傳達設計系", company: "明志科技大學" },
    { id: 'edu2', period: "2015 - 2018", role: "廣告設計科", company: "臺北市立士林高級商業職業學校" }
  ],
  skills: [
    { id: 'sk1', title: "品牌視覺識別 (VI)", detail: "標誌設計、色彩規範、品牌應用延伸。" },
    { id: 'sk2', title: "電商視覺設計", detail: "官網 Banner、社群圖文、素材規劃。" },
    { id: 'sk3', title: "平面排版印刷", detail: "百貨專刊設計、大型輸出、印刷管控。" },
    { id: 'sk4', title: "影像企劃與剪輯", detail: "產品攝影、短影音剪輯、動態視覺。" },
    { id: 'sk5', title: "AI 數位工作流", detail: "提示詞工程、AI 生成影像與音效。" }
  ],
  projects: [
    { 
      id: 15, title: "2025 海報系列", subtitle: "平面印刷實驗", category: "平面設計", tags: ["海報", "印刷"], 
      image: "https://images.unsplash.com/photo-1558591710-4b4a1ae0f04d?q=80&w=1000&auto=format&fit=crop",
      gallery: [
        "https://images.unsplash.com/photo-1558591710-4b4a1ae0f04d?q=80&w=1000&auto=format&fit=crop",
        "https://images.unsplash.com/photo-1561070791-2526d30994b5?q=80&w=1000&auto=format&fit=crop",
        "https://images.unsplash.com/photo-1541701494587-cb58502866ab?q=80&w=1000&auto=format&fit=crop"
      ]
    },
    { 
      id: 12, title: "鴻海科技集團", subtitle: "供應商責任報告書", category: "企業專案", tags: ["ESG", "PDF預覽"], 
      image: "https://images.unsplash.com/photo-1486406146926-c627a92ad1ab?q=80&w=1000&auto=format&fit=crop",
      pdfUrl: "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf"
    },
    { 
      id: 10, title: "天母 SOGO 專刊", subtitle: "2024 化妝品特輯", category: "排版設計", tags: ["百貨", "3D翻書"], 
      image: "https://images.unsplash.com/photo-1522335789203-aabd1fc54bc9?q=80&w=1000&auto=format&fit=crop",
      gallery: [
        "https://images.unsplash.com/photo-1522335789203-aabd1fc54bc9?q=80&w=1000&auto=format&fit=crop",
        "https://images.unsplash.com/photo-1586717791821-3f44a563eb4c?q=80&w=1000&auto=format&fit=crop"
      ]
    }
  ]
};

const safeStr = (val) => (typeof val === 'string' || typeof val === 'number' ? val : '');

// ==========================================
// 🏗️ 純淨展示組件 (Simple Gallery Viewer)
// ==========================================
const SimpleGalleryViewer = ({ images }) => {
  return (
    <div className="w-full space-y-12 py-4">
      {(images || []).map((url, idx) => (
        <div key={idx} className="w-full bg-white overflow-hidden rounded-sm shadow-md ring-1 ring-zinc-100 group/image">
          <img 
            src={url} 
            className="w-full h-auto object-cover transition-transform duration-700 group-hover/image:scale-[1.02]" 
            alt={`Work ${idx + 1}`} 
          />
        </div>
      ))}
      <div className="py-20 text-center">
         <p className="text-[10px] font-black uppercase tracking-[0.5em] text-zinc-300">End of Content</p>
      </div>
    </div>
  );
};

// ==========================================
// 🏗️ PDF 預覽組件
// ==========================================
const PdfViewer = ({ url }) => (
  <div className="w-full h-full flex flex-col gap-4 font-sans">
    <div className="flex-1 bg-zinc-100 rounded-sm overflow-hidden border border-zinc-200 shadow-inner relative min-h-[550px]">
      <iframe src={url} className="w-full h-full border-none" title="PDF Preview" />
      <div className="absolute top-2 right-2 flex gap-2">
         <a href={url} target="_blank" rel="noopener noreferrer" className="p-2 bg-white/90 text-zinc-900 rounded-full shadow-md hover:bg-red-600 hover:text-white transition-all"><ExternalLink size={16} /></a>
      </div>
    </div>
    <div className="flex items-center justify-between p-4 bg-zinc-50 border border-zinc-100 rounded-sm">
      <div className="flex items-center gap-3">
        <div className="p-2 bg-red-100 text-red-600 rounded-sm"><FileText size={20} /></div>
        <div><p className="text-xs font-black uppercase text-zinc-900">Document Review Mode</p><p className="text-[10px] text-zinc-400">您可以直接在視窗中滾動閱讀完整內容。</p></div>
      </div>
      <a href={url} download className="flex items-center gap-2 bg-zinc-900 text-white px-5 py-2 rounded-full text-[10px] font-black uppercase tracking-widest hover:bg-red-600 transition-all shadow-md"><Download size={14} /> Download PDF</a>
    </div>
  </div>
);

// ==========================================
// 🏗️ 翻頁動畫組件 (Magazine Viewer)
// ==========================================
const MagazineViewer = ({ images }) => {
  const [currentPage, setCurrentPage] = useState(0);
  const [isFlipping, setIsFlipping] = useState(false);
  const [flipDirection, setFlipDirection] = useState('next');
  const totalPages = images?.length || 0;
  if (totalPages === 0) return null;

  const flip = (dir) => {
    if (isFlipping) return;
    if (dir === 'next' && currentPage < totalPages - 2) {
      setFlipDirection('next'); setIsFlipping(true);
      setTimeout(() => { setCurrentPage(p => p + 2); setIsFlipping(false); }, 600);
    } else if (dir === 'prev' && currentPage > 0) {
      setFlipDirection('prev'); setIsFlipping(true);
      setTimeout(() => { setCurrentPage(p => p - 2); setIsFlipping(false); }, 600);
    }
  };

  return (
    <div className="flex flex-col items-center gap-8 w-full py-4 font-sans select-none">
      <div className="relative group w-full aspect-[3/2] flex items-center justify-center perspective-[2000px]">
        <div className="absolute inset-0 bg-zinc-100 rounded-sm border border-zinc-200 opacity-50 scale-95 flex"><div className="w-1/2 border-r border-zinc-300"></div><div className="w-1/2"></div></div>
        <div className="relative z-20 w-[95%] h-[95%] flex shadow-2xl bg-white preserve-3d transition-transform duration-500">
          <div className="w-1/2 h-full bg-white overflow-hidden relative border-r border-zinc-200 z-10">
            <img src={images[currentPage]} className="w-full h-full object-cover" alt="Left" />
            <div className="absolute right-0 top-0 bottom-0 w-10 bg-gradient-to-l from-black/10 to-transparent pointer-events-none" />
          </div>
          <div className="w-1/2 h-full bg-white overflow-hidden relative z-10">
            <img src={images[currentPage + 1] || images[currentPage]} className="w-full h-full object-cover" alt="Right" />
            <div className="absolute left-0 top-0 bottom-0 w-10 bg-gradient-to-r from-black/10 to-transparent pointer-events-none" />
          </div>
          {isFlipping && (
            <div className={`absolute top-0 bottom-0 w-1/2 h-full z-30 preserve-3d origin-right ${flipDirection === 'next' ? 'right-1/2' : 'left-1/2'}`} style={{ transformOrigin: flipDirection === 'next' ? 'right' : 'left', animation: flipDirection === 'next' ? 'flip-forward 0.6s ease-in-out forwards' : 'flip-backward 0.6s ease-in-out forwards' }}>
              <div className="absolute inset-0 backface-hidden z-20"><img src={flipDirection === 'next' ? images[currentPage + 1] : images[currentPage]} className="w-full h-full object-cover border-l border-zinc-100" /></div>
              <div className="absolute inset-0 backface-hidden z-10 rotate-y-180 bg-white"><img src={flipDirection === 'next' ? images[currentPage + 2] : images[currentPage - 1]} className="w-full h-full object-cover border-r border-zinc-100" /></div>
            </div>
          )}
        </div>
        <button onClick={() => flip('prev')} disabled={currentPage === 0} className={`absolute left-[-20px] z-40 p-3 bg-white text-zinc-900 rounded-full shadow-xl transition-all hover:bg-red-600 hover:text-white disabled:opacity-0`}><ChevronLeft size={24} /></button>
        <button onClick={() => flip('next')} disabled={currentPage >= totalPages - 2} className={`absolute right-[-20px] z-40 p-3 bg-white text-zinc-900 rounded-full shadow-xl transition-all hover:bg-red-600 hover:text-white disabled:opacity-0`}><ChevronRight size={24} /></button>
        <style>{`.preserve-3d{transform-style:preserve-3d;}.backface-hidden{backface-visibility:hidden;}.rotate-y-180{transform:rotateY(180deg);}@keyframes flip-forward{from{transform:rotateY(0);}to{transform:rotateY(-180deg);}}@keyframes flip-backward{from{transform:rotateY(0);}to{transform:rotateY(180deg);}}`}</style>
      </div>
      <div className="flex flex-col items-center gap-4">
        <p className="text-[10px] font-black uppercase tracking-[0.3em] text-zinc-900">{currentPage + 1}-{Math.min(currentPage + 2, totalPages)} / {totalPages} Pages</p>
        <div className="flex gap-1.5">{Array.from({ length: Math.ceil(totalPages / 2) }).map((_, i) => (<div key={i} className={`h-1 rounded-full transition-all duration-500 ${i === Math.floor(currentPage / 2) ? 'w-6 bg-red-600' : 'w-2 bg-zinc-200'}`} />))}</div>
      </div>
    </div>
  );
};

// ==========================================
// 🏗️ IG 貼文樣式 Mockup
// ==========================================
const SocialPostMockup = ({ images, profile }) => {
  const [slide, setSlide] = useState(0);
  const total = images?.length || 0;
  return (
    <div className="w-full max-w-[480px] mx-auto bg-white rounded-xl shadow-2xl border border-zinc-100 overflow-hidden font-sans">
      <div className="flex items-center justify-between p-3 border-b border-zinc-50">
        <div className="flex items-center gap-3">
          <div className="w-8 h-8 rounded-full bg-gradient-to-tr from-yellow-400 via-red-500 to-purple-600 p-[2px]"><div className="w-full h-full rounded-full bg-zinc-200 border-2 border-white flex items-center justify-center text-[10px] font-bold text-zinc-400">WP</div></div>
          <div><p className="text-[11px] font-bold tracking-tight">{safeStr(profile.nameEn)}</p><p className="text-[9px] text-zinc-400">{safeStr(profile.location)}</p></div>
        </div>
        <MoreHorizontal size={16} className="text-zinc-400" />
      </div>
      <div className="relative aspect-square bg-zinc-100 group">
        <img src={images[slide]} className="w-full h-full object-cover" alt="Slide" />
        {total > 1 && (<><button onClick={() => setSlide(s => Math.max(0, s - 1))} className={`absolute left-2 top-1/2 -translate-y-1/2 p-1.5 bg-white/90 rounded-full shadow-md ${slide === 0 ? 'opacity-0' : 'opacity-100'}`}><ChevronLeft size={16}/></button><button onClick={() => setSlide(s => Math.min(total - 1, s + 1))} className={`absolute right-2 top-1/2 -translate-y-1/2 p-1.5 bg-white/90 rounded-full shadow-md ${slide === total - 1 ? 'opacity-0' : 'opacity-100'}`}><ChevronRight size={16}/></button><div className="absolute top-3 right-3 bg-black/60 px-2 py-1 rounded-full text-[9px] text-white font-bold">{slide + 1}/{total}</div></>)}
      </div>
      <div className="p-4 space-y-3">
        <div className="flex items-center justify-between"><div className="flex items-center gap-4"><Heart size={20} className="hover:text-red-500 cursor-pointer" /><MessageCircle size={20}/><Send size={20}/></div><Bookmark size={20}/></div>
        <div className="flex justify-center gap-1">{images.map((_, i) => (<div key={i} className={`w-1.5 h-1.5 rounded-full ${i === slide ? 'bg-red-600 scale-125' : 'bg-zinc-200'}`} />))}</div>
        <div className="space-y-1"><p className="text-[11px] font-bold">1,234 likes</p><p className="text-[11px] leading-relaxed"><span className="font-bold mr-2">{safeStr(profile.nameEn)}</span>Visual identity & Social media campaign. #design #2026</p></div>
      </div>
    </div>
  );
};

// ==========================================
// 🏗️ 主程式 App
// ==========================================
const App = () => {
  const [user, setUser] = useState(null);
  const [data, setData] = useState(INITIAL_DATA);
  const [loading, setLoading] = useState(true);
  const [activeCategory, setActiveCategory] = useState('全部');
  const [selectedProject, setSelectedProject] = useState(null);
  const [showAdmin, setShowAdmin] = useState(false);
  const [isSaving, setIsSaving] = useState(false);

  useEffect(() => {
    const init = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) await signInWithCustomToken(auth, __initial_auth_token);
      else await signInAnonymously(auth);
      onAuthStateChanged(auth, setUser);
    };
    init();
  }, []);

  useEffect(() => {
    if (!user) return;
    const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'portfolio_v5', 'content');
    return onSnapshot(docRef, (snap) => {
      if (snap.exists()) setData(snap.data()); else setDoc(docRef, INITIAL_DATA);
      setLoading(false);
    });
  }, [user]);

  const saveToCloud = async (newData) => {
    if (!user) return; setIsSaving(true);
    try { await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'portfolio_v5', 'content'), newData); setIsSaving(false); setShowAdmin(false); } 
    catch (e) { console.error(e); setIsSaving(false); }
  };

  const projects = data?.projects || [];
  const categories = ['全部', ...new Set(projects.map(p => p.category))];
  const filtered = activeCategory === '全部' ? projects : projects.filter(p => p.category === activeCategory);

  if (loading) return <div className="h-screen flex items-center justify-center text-red-600 font-bold font-sans animate-pulse tracking-[0.3em]">INITIALIZING EXPERIENCE...</div>;

  return (
    <div className="min-h-screen bg-[#FDFDFD] text-zinc-900 selection:bg-red-500 selection:text-white font-sans antialiased">
      {/* Navbar */}
      <nav className="fixed w-full z-40 bg-white/70 backdrop-blur-xl border-b border-zinc-100 px-6">
        <div className="max-w-7xl mx-auto h-16 flex justify-between items-center">
          <div className="font-black text-xl tracking-tighter flex items-center gap-1">DESIGNER<span className="text-red-600">.</span></div>
          <div className="flex items-center gap-6">
            <div className="hidden md:flex gap-6 text-[10px] font-bold uppercase tracking-widest text-zinc-400">
              <a href="#about" className="hover:text-red-600 transition-colors">Profile</a><a href="#work" className="hover:text-red-600 transition-colors">Works</a><a href="#contact" className="hover:text-red-600 transition-colors">Contact</a>
            </div>
            <button onClick={() => setShowAdmin(!showAdmin)} className={`p-2 rounded-full transition-colors ${showAdmin ? 'bg-red-600 text-white' : 'bg-zinc-100 hover:bg-zinc-200 text-zinc-500'}`}><Settings size={16} /></button>
          </div>
        </div>
      </nav>

      {/* Admin CMS */}
      {showAdmin && (
        <div className="fixed inset-0 z-50 bg-[#F4F4F5] pt-20 overflow-y-auto px-6 pb-20 font-sans">
          <div className="max-w-5xl mx-auto">
             <div className="flex justify-between items-center mb-8 bg-white p-6 border rounded-sm shadow-sm">
                <div><h2 className="text-2xl font-black uppercase text-red-600">線上管理控制台</h2><p className="text-[10px] font-bold text-zinc-400 mt-1">Portfolio CMS v5.4 (Pure Stack UI)</p></div>
                <div className="flex gap-3"><button onClick={() => saveToCloud(data)} disabled={isSaving} className="flex items-center gap-2 bg-red-600 text-white px-8 py-3 rounded-full text-xs font-black uppercase hover:bg-red-700 disabled:opacity-50">{isSaving ? <Loader2 className="animate-spin" size={14} /> : <Save size={14} />} 儲存發布</button><button onClick={() => setShowAdmin(false)} className="p-3 bg-zinc-100 rounded-full hover:bg-zinc-200"><X size={20}/></button></div>
             </div>
             <div className="space-y-10">
                <section className="bg-white p-8 border shadow-sm">
                   <h3 className="text-xs font-black uppercase text-zinc-900 mb-8 border-l-4 border-red-600 pl-4">基本資料 Profile</h3>
                   <div className="grid md:grid-cols-2 gap-8"><div className="space-y-4"><input type="text" value={data.profile?.name || ''} onChange={(e) => setData({...data, profile: {...data.profile, name: e.target.value}})} className="w-full bg-zinc-50 border-b p-3 font-bold" /><input type="text" value={data.profile?.nameEn || ''} onChange={(e) => setData({...data, profile: {...data.profile, nameEn: e.target.value}})} className="w-full bg-zinc-50 border-b p-3 font-bold" /></div><textarea rows="5" value={data.profile?.intro || ''} onChange={(e) => setData({...data, profile: {...data.profile, intro: e.target.value}})} className="w-full bg-zinc-50 border p-4 text-sm" /></div>
                </section>
                <section className="bg-white p-8 border shadow-sm">
                   <div className="flex justify-between items-center mb-10 border-l-4 border-red-600 pl-4"><h3 className="text-xs font-black uppercase text-zinc-900">作品藝廊 Works Gallery</h3><button onClick={() => setData({...data, projects: [{id: Date.now(), title: "新專案", category: "平面設計", image: "", gallery: [], pdfUrl: ""}, ...(data.projects || [])]})} className="flex items-center gap-2 bg-zinc-900 text-white text-[10px] font-black px-6 py-2 rounded-full hover:bg-red-600 transition-all"><Plus size={14}/> 新增作品</button></div>
                   <div className="space-y-12">
                      {(data.projects || []).map((proj, idx) => (
                        <div key={proj.id} className="p-8 border border-zinc-100 rounded-sm bg-[#FCFCFC] relative group/card hover:border-red-200 transition-all">
                           <button onClick={() => setData({...data, projects: data.projects.filter(p => p.id !== proj.id)})} className="absolute top-4 right-4 text-zinc-300 hover:text-red-600"><Trash2 size={20}/></button>
                           <div className="grid lg:grid-cols-12 gap-8">
                              <div className="lg:col-span-3">
                                 <div className="aspect-[3/4] bg-zinc-200 rounded-sm overflow-hidden mb-4 ring-1 ring-zinc-100 shadow-inner">{proj.image ? <img src={proj.image} className="w-full h-full object-cover" /> : <div className="w-full h-full flex items-center justify-center text-zinc-400"><ImageIcon size={32} /></div>}</div>
                                 <input type="text" value={proj.image || ''} placeholder="封面圖網址" onChange={(e) => { const newP = [...data.projects]; newP[idx].image = e.target.value; setData({...data, projects: newP}); }} className="w-full text-[9px] p-2 bg-white border" />
                              </div>
                              <div className="lg:col-span-4 space-y-4">
                                 <input type="text" value={proj.title || ''} placeholder="專案名稱" onChange={(e) => { const newP = [...data.projects]; newP[idx].title = e.target.value; setData({...data, projects: newP}); }} className="w-full text-lg font-black border-b bg-transparent" />
                                 <select value={proj.category} onChange={(e) => { const newP = [...data.projects]; newP[idx].category = e.target.value; setData({...data, projects: newP}); }} className="w-full text-xs font-bold p-2 bg-white border border-zinc-200">
                                    <option value="平面設計">平面設計 (純淨並排)</option>
                                    <option value="品牌設計">品牌設計 (IG 輪播樣式)</option>
                                    <option value="排版設計">排版設計 (3D 翻頁樣式)</option>
                                    <option value="企業專案">企業專案 (3D 翻頁樣式)</option>
                                 </select>
                                 <div className="pt-4"><label className="text-[10px] font-black uppercase text-red-600 flex items-center gap-2"><FileText size={14}/> PDF 預覽網址</label><input type="text" value={proj.pdfUrl || ''} placeholder="https://.../report.pdf" onChange={(e) => { const newP = [...data.projects]; newP[idx].pdfUrl = e.target.value; setData({...data, projects: newP}); }} className="w-full text-[10px] p-2 bg-red-50 border outline-none font-mono" /></div>
                              </div>
                              <div className="lg:col-span-5 border-l pl-8">
                                 <label className="text-[10px] font-black uppercase text-zinc-900 block mb-4 flex items-center justify-between">多圖藝廊 ({proj.gallery?.length || 0}) <button onClick={() => { const newP = [...data.projects]; newP[idx].gallery = [...(newP[idx].gallery || []), ""]; setData({...data, projects: newP}); }} className="text-red-600 flex items-center gap-1 hover:underline"><Plus size={12}/> 新增一張圖</button></label>
                                 <div className="space-y-3 max-h-80 overflow-y-auto pr-2 no-scrollbar">
                                    {(proj.gallery || []).map((url, gIdx) => (
                                      <div key={gIdx} className="flex gap-3 items-center bg-white p-2 border border-zinc-100 rounded-sm group/imgedit">
                                         <div className="w-12 h-12 bg-zinc-100 rounded-sm overflow-hidden shrink-0 border border-zinc-200">{url ? <img src={url} className="w-full h-full object-cover" /> : <ImageIcon size={16} className="m-auto mt-4 text-zinc-300" />}</div>
                                         <input type="text" value={url} placeholder="貼上圖片連結" onChange={(e) => { const newP = [...data.projects]; newP[idx].gallery[gIdx] = e.target.value; setData({...data, projects: newP}); }} className="w-full text-[10px] bg-transparent outline-none font-mono" />
                                         <button onClick={() => { const newP = [...data.projects]; newP[idx].gallery.splice(gIdx, 1); setData({...data, projects: newP}); }} className="text-zinc-300 hover:text-red-600 transition-colors"><Trash2 size={14}/></button>
                                      </div>
                                    ))}
                                 </div>
                              </div>
                           </div>
                        </div>
                      ))}
                   </div>
                </section>
             </div>
          </div>
        </div>
      )}

      {/* Main Content */}
      <header id="about" className="pt-32 pb-20 px-6 max-w-7xl mx-auto border-b border-zinc-100 font-sans">
        <div className="grid lg:grid-cols-12 gap-12 lg:gap-24">
          <div className="lg:col-span-4 space-y-8">
            <div>
              <p className="text-xs font-bold uppercase tracking-[0.3em] text-zinc-400 mb-4 flex items-center gap-2">Graphic Designer <span className="w-1 h-1 bg-red-500 rounded-full"></span> Visual Artist</p>
              <h1 className="text-6xl md:text-7xl font-black tracking-tighter leading-none mb-6 relative">{safeStr(data.profile?.name)}<br /><span className="text-4xl md:text-5xl">{safeStr(data.profile?.nameEn)}</span><span className="absolute -right-4 bottom-2 w-3 h-3 bg-red-600 rounded-full animate-pulse shadow-[0_0_10px_rgba(239,68,68,0.5)]"></span></h1>
              <div className="flex flex-wrap items-center gap-4 text-sm text-zinc-500 font-medium"><MapPin size={14} className="text-red-500" /> {safeStr(data.profile?.location)}</div>
            </div>
            <p className="text-lg text-zinc-600 leading-relaxed italic">{safeStr(data.profile?.intro)}</p>
            <div className="pt-4 flex items-center gap-6">
              <a href={`mailto:${data.profile?.email}`} className="text-sm font-bold border-b-2 border-red-600 pb-1 hover:text-red-600 transition-all uppercase tracking-widest inline-flex items-center gap-2 group">{safeStr(data.profile?.email)}<ArrowUpRight size={14} className="group-hover:translate-x-1 group-hover:-translate-y-1 transition-transform" /></a>
              <a href={safeStr(data.profile?.instagram)} target="_blank" rel="noopener noreferrer" className="p-2 border border-zinc-100 rounded-full hover:bg-zinc-100 transition-colors text-zinc-400 hover:text-red-600"><Instagram size={20}/></a>
            </div>
          </div>
          <div className="lg:col-span-8 grid md:grid-cols-2 gap-12 border-l border-zinc-100 lg:pl-12">
            <div className="space-y-10">
              <div className="flex items-center gap-3 border-b border-zinc-900 pb-2 mb-8 text-red-600"><Briefcase size={16} /><h2 className="text-xs font-black uppercase text-zinc-900">工作經歷 / Work</h2></div>
              <div className="relative">{(data.experience || []).map((exp, idx) => (<div key={exp.id} className="relative pl-8 pb-10 group"><div className="absolute left-[-4.5px] top-1.5 w-2.5 h-2.5 rounded-full bg-white border-2 border-red-500 group-hover:bg-red-500 transition-all shadow-[0_0_0_0_rgba(239,68,68,0.2)] group-hover:shadow-[0_0_0_6px_rgba(239,68,68,0.1)]"></div><p className="text-[10px] font-bold text-zinc-400 mb-2 tracking-widest uppercase">{safeStr(exp.period)}</p><h3 className="font-bold text-base text-zinc-900 group-hover:text-red-600 transition-colors mb-1">{safeStr(exp.company)}</h3><p className="text-sm font-medium text-zinc-600 mb-3">{safeStr(exp.role)}</p><p className="text-xs text-zinc-400 leading-relaxed max-w-sm whitespace-pre-line">{safeStr(exp.desc)}</p></div>))}</div>
            </div>
            <div className="space-y-12">
              <div><div className="flex items-center gap-3 border-b border-zinc-900 pb-2 mb-8 text-red-600"><GraduationCap size={16} /><h2 className="text-xs font-black uppercase text-zinc-900">學歷 / Education</h2></div><div className="relative">{(data.education || []).map((edu, idx) => (<div key={edu.id} className="relative pl-8 pb-8 group"><div className="absolute left-[-4.5px] top-1.5 w-2.5 h-2.5 rounded-full bg-white border-2 border-red-500 group-hover:bg-red-500 transition-all shadow-[0_0_0_0_rgba(239,68,68,0.2)] group-hover:shadow-[0_0_0_6px_rgba(239,68,68,0.1)]"></div><p className="text-[10px] font-bold text-zinc-400 mb-1 tracking-widest uppercase">{safeStr(edu.period)}</p><h3 className="font-bold text-base text-zinc-900 group-hover:text-red-600 transition-colors mb-1">{safeStr(edu.company)}</h3><p className="text-sm font-medium text-zinc-600">{safeStr(edu.role)}</p></div>))}</div></div>
              <div><div className="flex items-center gap-3 border-b border-zinc-900 pb-2 mb-8 text-red-600"><PenTool size={16} /><h2 className="text-xs font-black uppercase text-zinc-900">專業技能 / Skills</h2></div><div className="space-y-4">{(data.skills || []).map(skill => (<div key={skill.id} className="group/skill font-sans"><h4 className="text-[11px] font-black uppercase tracking-widest text-zinc-900 mb-1 flex items-center gap-2 group-hover/skill:text-red-600 transition-colors"><span className="w-1 h-1 bg-red-500 rounded-full opacity-0 group-hover/skill:opacity-100 transition-opacity"></span>{safeStr(skill.title)}</h4><p className="text-[10px] text-zinc-500 leading-relaxed pl-3 border-l border-zinc-100 group-hover/skill:border-red-200 transition-colors">{safeStr(skill.detail)}</p></div>))}</div></div>
            </div>
          </div>
        </div>
      </header>

      {/* Filter Grid */}
      <div id="work" className="sticky top-16 z-30 bg-[#FDFDFD]/90 backdrop-blur-sm border-b border-zinc-100 mb-12 px-6">
        <div className="max-w-7xl mx-auto py-6 flex justify-between items-center">
          <h2 className="text-xs font-black uppercase tracking-[0.3em]">Selected Works</h2>
          <div className="flex gap-6 overflow-x-auto no-scrollbar font-sans">{categories.map(cat => (<button key={cat} onClick={() => setActiveCategory(cat)} className={`whitespace-nowrap text-[10px] font-bold uppercase tracking-widest transition-all ${activeCategory === cat ? 'text-red-600 border-b-2 border-red-600 pb-1' : 'text-zinc-400 hover:text-zinc-900'}`}>{safeStr(cat)}</button>))}</div>
        </div>
      </div>

      <main className="px-6 max-w-7xl mx-auto pb-32 font-sans">
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-x-12 gap-y-20">
          {filtered.map((project) => (
            <div key={project.id} className="group cursor-pointer" onClick={() => setSelectedProject(project)}>
              <div className={`relative aspect-[3/4] overflow-hidden mb-6 rounded-sm transition-all duration-700 ease-out group-hover:shadow-2xl group-hover:-translate-y-2 shadow-sm bg-white ring-1 ring-black/5`}>
                {(project.category !== "品牌設計" && project.category !== "平面設計") && (<div className="absolute left-0 top-0 bottom-0 w-6 bg-gradient-to-r from-black/10 via-black/5 to-transparent z-10" />)}
                <div className="absolute inset-0 bg-gradient-to-tr from-transparent via-white/5 to-white/10 z-10 opacity-0 group-hover:opacity-100 duration-700" />
                <img src={project.image} alt={safeStr(project.title)} className={`w-full h-full object-cover transition-all duration-1000 ease-out grayscale-[0.3] group-hover:grayscale-0 group-hover:scale-110`} />
                <div className="absolute top-4 left-4 z-20"><div className="w-1.5 h-1.5 bg-red-600 rounded-full opacity-0 group-hover:opacity-100 transition-all duration-500 scale-0 group-hover:scale-100" /></div>
                <div className="absolute bottom-4 right-4 opacity-0 group-hover:opacity-100 transition-opacity flex gap-2 z-20">
                   {project.pdfUrl && <div className="bg-red-600 p-1.5 rounded-full shadow-lg text-white"><FileText size={16} /></div>}
                   {project.category === "品牌設計" && project.gallery?.length > 0 && <div className="bg-white/90 p-1.5 rounded-full shadow-lg text-red-600"><Instagram size={16} /></div>}
                   {(project.category === "排版設計" || project.category === "企業專案") && project.gallery?.length > 0 && <div className="bg-white/90 p-1.5 rounded-full shadow-lg text-red-600"><BookOpen size={16} /></div>}
                   {project.category === "平面設計" && project.gallery?.length > 0 && <div className="bg-white/90 p-1.5 rounded-full shadow-lg text-red-600"><Eye size={16} /></div>}
                </div>
              </div>
              <div className="flex justify-between items-end border-b border-transparent group-hover:border-red-200 pb-2 transition-all px-1">
                <div>
                  <div className="flex gap-2 mb-2">{(project.tags || []).map(tag => <span key={tag} className="text-[9px] uppercase tracking-wider bg-zinc-100 px-2 py-0.5 rounded-sm font-bold text-zinc-500 group-hover:text-red-500 transition-colors">{safeStr(tag)}</span>)}</div>
                  <h3 className="text-xl font-bold tracking-tight group-hover:text-red-600 transition-colors">{safeStr(project.title)}</h3>
                  <p className="text-sm text-zinc-400 font-medium">{safeStr(project.subtitle)}</p>
                </div>
                <div className="w-6 h-6 rounded-full flex items-center justify-center opacity-0 group-hover:opacity-100 group-hover:text-red-600 transition-all"><ArrowUpRight size={18} /></div>
              </div>
            </div>
          ))}
        </div>
      </main>

      <section id="contact" className="py-32 px-6 max-w-7xl mx-auto border-t text-center font-sans">
        <h2 className="text-xs font-black uppercase text-zinc-400 flex justify-center items-center gap-2 mb-12"><span className="w-8 h-px bg-zinc-100"></span>Get in touch<span className="w-8 h-px bg-zinc-100"></span></h2>
        <div className="flex flex-col items-center gap-6"><a href={`mailto:${data.profile.email}`} className="text-2xl md:text-4xl font-bold tracking-tighter hover:text-red-600 transition-colors relative group">{safeStr(data.profile.email)}<span className="absolute -right-3 top-1 w-2 h-2 bg-red-600 rounded-full opacity-0 group-hover:opacity-100 transition-opacity"></span></a></div>
      </section>

      {/* Modal View (支援翻頁, IG, PDF, 及純淨並排) */}
      {selectedProject && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-white/98 backdrop-blur-xl" onClick={() => setSelectedProject(null)}>
          <div className="w-full max-w-6xl max-h-[95vh] overflow-y-auto relative no-scrollbar bg-white p-8 md:p-12 shadow-2xl rounded-sm border border-zinc-100" onClick={e => e.stopPropagation()}>
            <button onClick={() => setSelectedProject(null)} className="fixed top-8 right-8 z-50 p-2 bg-red-600 text-white rounded-full hover:scale-110 transition-all shadow-lg"><X size={24}/></button>
            <div className="grid lg:grid-cols-12 gap-12">
              <div className="lg:col-span-4 font-sans">
                <span className="text-[10px] uppercase font-black text-red-500 mb-4 block">Project Detail / {safeStr(selectedProject.category)}</span>
                <h2 className="text-5xl font-black mb-6 tracking-tighter leading-none">{safeStr(selectedProject.title)}</h2>
                <p className="text-xl text-zinc-500 mb-10 font-medium">{safeStr(selectedProject.subtitle)}</p>
                <div className="border-t border-zinc-100 pt-8 space-y-6">
                  <div>
                    <h4 className="font-black mb-3 uppercase text-[10px] tracking-widest text-red-600">Project Scope</h4>
                    <p className="text-zinc-600 text-sm leading-relaxed">
                      {selectedProject.pdfUrl ? "完整 PDF 文件預覽模式。" : (selectedProject.category === "平面設計" ? "此專案以純淨並排方式展示系列作品，確保設計細節與排版比例的完整呈現。" : (selectedProject.category === "品牌設計" ? "社群體系視覺規劃與溝通。" : "3D 翻頁互動版面展示。"))}
                    </p>
                  </div>
                  {(selectedProject.gallery?.length > 0 || selectedProject.pdfUrl) && (
                    <div className="p-4 bg-red-50/50 border border-red-100 rounded-sm font-sans font-bold">
                       <p className="text-[10px] font-bold text-red-600 uppercase mb-1 flex items-center gap-2">
                         {selectedProject.pdfUrl ? <FileText size={12}/> : (selectedProject.category === "平面設計" ? <Eye size={12}/> : <Layers size={12}/>)} 
                         {selectedProject.category === "平面設計" ? "Linear View Active" : "Interactive Mode Active"}
                       </p>
                    </div>
                  )}
                </div>
              </div>
              <div className="lg:col-span-8 flex items-center justify-center min-h-[550px]">
                {selectedProject.pdfUrl ? (
                  <PdfViewer url={selectedProject.pdfUrl} />
                ) : (
                  selectedProject.gallery?.length > 0 ? (
                    selectedProject.category === "品牌設計" ? (
                      <SocialPostMockup images={selectedProject.gallery} profile={data.profile} />
                    ) : (
                      selectedProject.category === "平面設計" ? (
                        <SimpleGalleryViewer images={selectedProject.gallery} />
                      ) : (
                        <MagazineViewer images={selectedProject.gallery} />
                      )
                    )
                  ) : (
                    <div className="w-full aspect-[4/5] bg-zinc-50 rounded-sm overflow-hidden ring-1 ring-zinc-100 shadow-2xl relative">
                       {(selectedProject.category !== "品牌設計" && selectedProject.category !== "平面設計") && <div className="absolute left-0 top-0 bottom-0 w-8 bg-gradient-to-r from-black/10 to-transparent z-10" />}
                       <img src={selectedProject.image} className="w-full h-full object-cover" alt="Detail" />
                    </div>
                  )
                )}
              </div>
            </div>
          </div>
        </div>
      )}

      <footer className="px-6 py-12 border-t border-zinc-100 text-center text-zinc-400 text-[9px] uppercase tracking-widest italic font-sans"><p>&copy; 2026 Designed for Professional Impact <span className="w-1 h-1 bg-red-500 inline-block mx-2 rounded-full"></span> Portfolio v5.4</p></footer>
    </div>
  );
};

export default App;