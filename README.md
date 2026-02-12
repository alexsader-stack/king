import React, { useState, useEffect, useRef } from 'react';
import { 
  PlusCircle, 
  FileText, 
  CheckCircle, 
  XCircle, 
  Download, 
  Trash2, 
  Award,
  Upload,
  User,
  Calendar,
  ClipboardList,
  LogOut,
  Copy,
  Check
} from 'lucide-react';

// --- Konfigurasi Logo ---
const LOGO_URL = "https://i.ibb.co/v4mSPhn/SARAWAK-SKILLS-MIRI-LOGO-copy.jpg"; 

const App = () => {
  // --- State ---
  const [students, setStudents] = useState([]);
  const [formData, setFormData] = useState({
    nama: '',
    jenisAmali: '',
    tarikh: new Date().toISOString().split('T')[0],
    markah: '',
    tahapPencapaian: '1', 
    komen: '',
    pdfFile: null
  });
  const [view, setView] = useState('dashboard'); 
  const [selectedStudent, setSelectedStudent] = useState(null);
  const [copied, setCopied] = useState(false);
  const fileInputRef = useRef(null);

  // --- Logik Penilaian ---
  const hitungStatus = (markah) => {
    return parseInt(markah) >= 60 ? 'TERAMPIL' : 'TIDAK TERAMPIL';
  };

  const dapatkanTahapTeks = (tahap) => {
    switch (tahap) {
      case '1': return 'Sangat Memuaskan';
      case '2': return 'Kurang Memuaskan';
      case '3': return 'Sangat Tidak Memuaskan';
      default: return '';
    }
  };

  // --- Fungsi Salin Kod (Untuk memudahkan user) ---
  const copyToClipboard = () => {
    const code = document.documentElement.outerHTML;
    const textArea = document.createElement("textarea");
    textArea.value = code;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch (err) {
      console.error('Gagal menyalin kod', err);
    }
    document.body.removeChild(textArea);
  };

  // --- Actions ---
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleFileChange = (e) => {
    const file = e.target.files[0];
    if (file && file.type === 'application/pdf') {
      setFormData(prev => ({ ...prev, pdfFile: file.name }));
    } else {
      alert("Sila muat naik fail dalam format PDF sahaja.");
      e.target.value = null;
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!formData.nama || !formData.markah) return;

    const newStudent = {
      id: Date.now(),
      ...formData,
      status: hitungStatus(formData.markah),
      tahapTeks: dapatkanTahapTeks(formData.tahapPencapaian)
    };

    setStudents([newStudent, ...students]);
    setFormData({
      nama: '',
      jenisAmali: '',
      tarikh: new Date().toISOString().split('T')[0],
      markah: '',
      tahapPencapaian: '1',
      komen: '',
      pdfFile: null
    });
    setView('dashboard');
  };

  const deleteStudent = (id) => {
    setStudents(students.filter(s => s.id !== id));
  };

  const printCertificate = (student) => {
    setSelectedStudent(student);
    setView('sijil');
    setTimeout(() => {
      window.print();
    }, 500);
  };

  // --- UI Components ---
  const Sidebar = () => (
    <div className="w-64 bg-slate-900 text-white min-h-screen p-6 hidden md:flex flex-col">
      <div className="flex flex-col items-center mb-10 text-center">
        <img src={LOGO_URL} alt="Logo" className="w-32 mb-4 rounded bg-white p-1" />
        <h1 className="text-sm font-bold">SARAWAK SKILLS MIRI</h1>
        <p className="text-xs text-slate-400 uppercase tracking-tighter">Jabatan Sistem Komputer</p>
      </div>
      
      <nav className="space-y-2 flex-1">
        <button 
          onClick={() => setView('dashboard')}
          className={`flex items-center w-full p-3 rounded-lg transition ${view === 'dashboard' ? 'bg-emerald-600 text-white' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}
        >
          <ClipboardList className="mr-3 w-5 h-5" /> Dashboard
        </button>
        <button 
          onClick={() => setView('tambah')}
          className={`flex items-center w-full p-3 rounded-lg transition ${view === 'tambah' ? 'bg-emerald-600 text-white' : 'text-slate-400 hover:bg-slate-800 hover:text-white'}`}
        >
          <PlusCircle className="mr-3 w-5 h-5" /> Penilaian Baru
        </button>
      </nav>

      <div className="mt-auto space-y-4 pt-6 border-t border-slate-800">
        <button 
          onClick={copyToClipboard}
          className="flex items-center w-full p-3 rounded-lg bg-slate-800 text-slate-300 hover:bg-slate-700 transition text-sm"
        >
          {copied ? <Check className="mr-3 w-5 h-5 text-emerald-500" /> : <Copy className="mr-3 w-5 h-5" />}
          {copied ? 'Kod Disalin!' : 'Salin Kod Sistem'}
        </button>
        <button className="flex items-center w-full p-3 text-slate-500 hover:text-red-400 transition text-sm">
          <LogOut className="mr-3 w-5 h-5" /> Log Keluar
        </button>
      </div>
    </div>
  );

  return (
    <div className="flex min-h-screen bg-slate-50 font-sans text-slate-900">
      {view !== 'sijil' && <Sidebar />}

      <main className={`flex-1 ${view === 'sijil' ? 'p-0' : 'p-4 md:p-8'} overflow-y-auto`}>
        
        {view === 'dashboard' && (
          <div className="max-w-6xl mx-auto">
            <header className="mb-8 flex flex-col md:flex-row md:items-end justify-between gap-4">
              <div>
                <h2 className="text-3xl font-bold text-slate-800">Penilaian Amali</h2>
                <p className="text-slate-500">Sistem Pengurusan Prestasi Pelajar Sarawak Skills</p>
              </div>
              <button 
                onClick={() => setView('tambah')}
                className="bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2.5 rounded-xl flex items-center shadow-lg shadow-emerald-100 transition font-semibold"
              >
                <PlusCircle className="mr-2 w-5 h-5" /> Tambah Rekod
              </button>
            </header>

            <div className="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
              <div className="overflow-x-auto">
                <table className="w-full text-left border-collapse">
                  <thead className="bg-slate-50 text-slate-500 uppercase text-[11px] font-bold tracking-wider">
                    <tr>
                      <th className="px-6 py-4">Nama Pelajar</th>
                      <th className="px-6 py-4">Amali & Tarikh</th>
                      <th className="px-6 py-4 text-center">Markah</th>
                      <th className="px-6 py-4">Status</th>
                      <th className="px-6 py-4 text-center">Tindakan</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-100">
                    {students.length === 0 ? (
                      <tr>
                        <td colSpan="5" className="px-6 py-16 text-center text-slate-400">
                          <div className="flex flex-col items-center">
                            <ClipboardList className="w-12 h-12 mb-2 opacity-20" />
                            <p className="italic">Belum ada rekod penilaian dimasukkan.</p>
                          </div>
                        </td>
                      </tr>
                    ) : (
                      students.map(s => (
                        <tr key={s.id} className="hover:bg-slate-50/50 transition">
                          <td className="px-6 py-4">
                            <div className="font-bold text-slate-800">{s.nama}</div>
                            <div className="text-[11px] text-slate-500 flex items-center mt-0.5">
                              <FileText className="w-3 h-3 mr-1" /> {s.pdfFile || 'Tiada PDF'}
                            </div>
                          </td>
                          <td className="px-6 py-4">
                            <div className="text-sm font-medium text-slate-700">{s.jenisAmali}</div>
                            <div className="text-[11px] text-slate-400 font-mono">{s.tarikh}</div>
                          </td>
                          <td className="px-6 py-4 text-center">
                            <span className="text-lg font-black text-slate-800">{s.markah}</span>
                            <span className="text-xs text-slate-400 ml-0.5">%</span>
                          </td>
                          <td className="px-6 py-4">
                            <div className={`inline-flex items-center px-2.5 py-0.5 rounded-full text-[10px] font-black tracking-widest ${s.status === 'TERAMPIL' ? 'bg-emerald-100 text-emerald-700' : 'bg-red-100 text-red-700'}`}>
                              {s.status === 'TERAMPIL' ? <CheckCircle className="w-3 h-3 mr-1" /> : <XCircle className="w-3 h-3 mr-1" />}
                              {s.status}
                            </div>
                          </td>
                          <td className="px-6 py-4">
                            <div className="flex justify-center space-x-1">
                              {s.status === 'TERAMPIL' && (
                                <button 
                                  onClick={() => printCertificate(s)}
                                  className="p-2 text-blue-600 hover:bg-blue-50 rounded-lg transition"
                                  title="Jana Sijil"
                                >
                                  <Award className="w-5 h-5" />
                                </button>
                              )}
                              <button 
                                onClick={() => deleteStudent(s.id)}
                                className="p-2 text-slate-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition"
                              >
                                <Trash2 className="w-5 h-5" />
                              </button>
                            </div>
                          </td>
                        </tr>
                      ))
                    )}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {view === 'tambah' && (
          <div className="max-w-xl mx-auto">
            <div className="bg-white p-8 rounded-3xl shadow-xl border border-slate-100">
              <div className="flex items-center justify-between mb-8">
                <h2 className="text-2xl font-bold text-slate-800">Borang Penilaian</h2>
                <div className="bg-emerald-50 text-emerald-600 p-2 rounded-xl">
                  <ClipboardList className="w-6 h-6" />
                </div>
              </div>

              <form onSubmit={handleSubmit} className="space-y-5">
                <div className="space-y-1.5">
                  <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Nama Pelajar</label>
                  <input 
                    type="text" name="nama" required
                    className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:bg-white outline-none transition"
                    placeholder="Masukkan nama penuh"
                    value={formData.nama} onChange={handleChange}
                  />
                </div>

                <div className="grid grid-cols-2 gap-4">
                  <div className="space-y-1.5">
                    <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Tarikh</label>
                    <input 
                      type="date" name="tarikh" required
                      className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition"
                      value={formData.tarikh} onChange={handleChange}
                    />
                  </div>
                  <div className="space-y-1.5">
                    <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Markah (0-100)</label>
                    <input 
                      type="number" name="markah" min="0" max="100" required
                      className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition"
                      value={formData.markah} onChange={handleChange}
                    />
                  </div>
                </div>

                <div className="space-y-1.5">
                  <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Jenis Amali</label>
                  <input 
                    type="text" name="jenisAmali" required
                    className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition"
                    placeholder="Contoh: Troubleshooting PC"
                    value={formData.jenisAmali} onChange={handleChange}
                  />
                </div>

                <div className="space-y-1.5">
                  <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Tahap Pencapaian</label>
                  <select 
                    name="tahapPencapaian" 
                    className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition appearance-none"
                    value={formData.tahapPencapaian} onChange={handleChange}
                  >
                    <option value="1">1. Sangat Memuaskan</option>
                    <option value="2">2. Kurang Memuaskan</option>
                    <option value="3">3. Sangat Tidak Memuaskan</option>
                  </select>
                </div>

                <div className="space-y-1.5">
                  <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Komen & Maklumbalas</label>
                  <textarea 
                    name="komen" rows="2"
                    className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition"
                    placeholder="Tulis komen pengajar di sini..."
                    value={formData.komen} onChange={handleChange}
                  ></textarea>
                </div>

                <div className="space-y-1.5">
                  <label className="text-xs font-bold text-slate-500 uppercase tracking-wider ml-1">Dokumen Amali (PDF)</label>
                  <div 
                    onClick={() => fileInputRef.current.click()}
                    className="w-full border-2 border-dashed border-slate-200 rounded-2xl p-4 flex flex-col items-center justify-center cursor-pointer hover:bg-slate-50 transition hover:border-emerald-300 group"
                  >
                    <Upload className="w-6 h-6 text-slate-300 group-hover:text-emerald-500 mb-1 transition" />
                    <span className="text-xs text-slate-500 font-medium">
                      {formData.pdfFile ? formData.pdfFile : "Pilih fail PDF amali"}
                    </span>
                    <input 
                      type="file" accept=".pdf" ref={fileInputRef} hidden
                      onChange={handleFileChange}
                    />
                  </div>
                </div>

                <div className="flex space-x-3 pt-4">
                  <button 
                    type="button" 
                    onClick={() => setView('dashboard')}
                    className="flex-1 px-4 py-3 text-slate-600 font-bold hover:bg-slate-100 rounded-2xl transition"
                  >
                    Batal
                  </button>
                  <button 
                    type="submit"
                    className="flex-1 bg-emerald-600 text-white px-4 py-3 rounded-2xl hover:bg-emerald-700 font-bold shadow-lg shadow-emerald-100 transition"
                  >
                    Simpan Rekod
                  </button>
                </div>
              </form>
            </div>
          </div>
        )}

        {view === 'sijil' && selectedStudent && (
          <div className="print-container bg-white min-h-screen flex items-center justify-center p-0 md:p-10">
            <div className="w-full max-w-4xl border-[20px] border-emerald-900 p-1 bg-white shadow-2xl">
              <div className="border-[4px] border-emerald-600 p-16 text-center bg-white relative overflow-hidden">
                
                <div className="absolute top-[-50px] left-[-50px] w-48 h-48 rounded-full bg-emerald-50 opacity-50"></div>
                <div className="absolute bottom-[-50px] right-[-50px] w-64 h-64 rounded-full bg-emerald-50 opacity-50"></div>

                <div className="flex justify-center mb-10 relative">
                  <img src={LOGO_URL} alt="Logo" className="h-32" />
                </div>

                <h1 className="text-5xl font-serif font-black text-emerald-900 mb-4 tracking-tighter uppercase">
                  Sijil Penghargaan
                </h1>
                <div className="w-24 h-1 bg-emerald-600 mx-auto mb-10"></div>

                <p className="text-xl mb-6 text-slate-600">Dengan penuh kebanggaan, Sarawak Skills Miri memperakui bahawa</p>
                <h2 className="text-5xl font-serif font-extrabold text-slate-900 mb-8 px-4 inline-block border-b-4 border-emerald-500 italic">
                  {selectedStudent.nama.toUpperCase()}
                </h2>

                <p className="max-w-2xl mx-auto text-xl leading-relaxed text-slate-700 mb-12">
                  Telah menyempurnakan penilaian amali bagi modul <br/> 
                  <span className="font-black text-slate-900">"{selectedStudent.jenisAmali}"</span> <br/>
                  dan mencapai keputusan <span className="font-black text-emerald-700">"{selectedStudent.status}"</span> <br/>
                  dengan prestasi <span className="font-bold underline italic text-emerald-800">{selectedStudent.tahapTeks}</span>.
                </p>

                <div className="grid grid-cols-2 gap-24 mt-20 relative">
                  <div className="text-center">
                    <div className="border-t-2 border-slate-300 pt-3">
                      <p className="text-sm font-bold text-slate-800 uppercase tracking-widest">Sarawak Skills Miri</p>
                      <p className="text-[10px] text-slate-500 font-bold uppercase tracking-widest mt-1">Sistem Komputer</p>
                    </div>
                  </div>
                  <div className="text-center">
                    <div className="border-t-2 border-slate-300 pt-3">
                      <p className="text-sm font-bold text-slate-800 uppercase tracking-widest">{selectedStudent.tarikh}</p>
                      <p className="text-[10px] text-slate-500 font-bold uppercase tracking-widest mt-1">Tarikh Penganugerahan</p>
                    </div>
                  </div>
                </div>

                <div className="mt-16 opacity-30">
                  <p className="text-[10px] font-mono tracking-widest uppercase">E-Certificate ID: SS-MIRI-{selectedStudent.id}</p>
                </div>
              </div>
            </div>
            
            <div className="fixed bottom-8 right-8 flex space-x-3 no-print">
              <button 
                onClick={() => setView('dashboard')}
                className="bg-slate-900 text-white px-8 py-4 rounded-2xl shadow-2xl hover:bg-black transition font-bold"
              >
                Tutup Sijil
              </button>
              <button 
                onClick={() => window.print()}
                className="bg-emerald-600 text-white px-8 py-4 rounded-2xl shadow-2xl hover:bg-emerald-700 transition font-bold flex items-center"
              >
                <Download className="mr-2 w-6 h-6" /> Cetak Sekarang
              </button>
            </div>
          </div>
        )}

      </main>

      <style>{`
        @media print {
          .no-print { display: none !important; }
          body { background: white !important; -webkit-print-color-adjust: exact; }
          main { padding: 0 !important; }
          .print-container { padding: 0 !important; margin: 0 !important; width: 100% !important; min-height: 100vh !important; display: flex !important; align-items: center !important; justify-content: center !important; }
          .print-container > div { box-shadow: none !important; border-width: 15px !important; }
        }
        select { background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='%2364748b'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M19 9l-7 7-7-7'%3E%3C/path%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 1rem center; background-size: 1em; }
      `}</style>
    </div>
  );
};

export default App;
