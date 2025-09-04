import React, { useEffect, useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import { Sun, Moon, Heart, Upload, Pencil, Trash2, Calendar, Image as ImageIcon, Video as VideoIcon, Settings, Music } from "lucide-react";

// Minimal helper UI (Tailwind-based) so it runs standalone in this preview.
const Button = ({ className = "", children, ...props }) => (
  <button
    className={`inline-flex items-center gap-2 rounded-2xl px-4 py-2 shadow-sm hover:shadow transition border border-black/5 disabled:opacity-50 ${className}`}
    {...props}
  >
    {children}
  </button>
);
const Card = ({ className = "", children }) => (
  <div className={`rounded-3xl shadow-md border border-black/5 ${className}`}>{children}</div>
);
const CardContent = ({ className = "", children }) => (
  <div className={`p-5 ${className}`}>{children}</div>
);
const Input = ({ className = "", ...props }) => (
  <input className={`w-full rounded-xl border border-black/10 px-3 py-2 outline-none focus:ring focus:ring-black/5 ${className}`} {...props} />
);
const Textarea = ({ className = "", ...props }) => (
  <textarea className={`w-full rounded-xl border border-black/10 px-3 py-2 outline-none focus:ring focus:ring-black/5 ${className}`} {...props} />
);
const Chip = ({ children }) => (
  <span className="text-xs px-2 py-1 rounded-full border border-black/10 bg-white/50 backdrop-blur shadow-sm">
    {children}
  </span>
);

// Types
const THEMES = {
  sun: {
    name: "Sol",
    bg: "from-amber-100 via-orange-100 to-rose-100",
    glow: "bg-gradient-to-br from-amber-300/30 via-orange-300/20 to-rose-300/10",
    text: "text-orange-800",
    accent: "bg-gradient-to-r from-amber-400 to-orange-400",
    icon: Sun,
  },
  moon: {
    name: "Lua",
    bg: "from-indigo-900 via-slate-900 to-black",
    glow: "bg-gradient-to-br from-indigo-500/10 via-sky-500/10 to-white/5",
    text: "text-indigo-100",
    accent: "bg-gradient-to-r from-indigo-400 to-sky-400",
    icon: Moon,
  },
};

function formatDuration(startDate) {
  if (!startDate) return { years: 0, months: 0, days: 0, text: "Defina a data do início do namoro" };
  const start = new Date(startDate);
  const now = new Date();

  let years = now.getFullYear() - start.getFullYear();
  let months = now.getMonth() - start.getMonth();
  let days = now.getDate() - start.getDate();

  if (days < 0) {
    const prevMonth = new Date(now.getFullYear(), now.getMonth(), 0).getDate();
    days += prevMonth;
    months -= 1;
  }
  if (months < 0) {
    months += 12;
    years -= 1;
  }

  const parts = [];
  if (years > 0) parts.push(`${years} ${years === 1 ? "ano" : "anos"}`);
  if (months > 0) parts.push(`${months} ${months === 1 ? "mês" : "meses"}`);
  parts.push(`${days} ${days === 1 ? "dia" : "dias"}`);

  return { years, months, days, text: parts.join(", ") };
}

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const raw = localStorage.getItem(key);
      return raw ? JSON.parse(raw) : initialValue;
    } catch (e) {
      return initialValue;
    }
  });
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (e) {
      // ignore
    }
  }, [key, value]);
  return [value, setValue];
}

export default function SabrinaLeilianeSite() {
  const [theme, setTheme] = useLocalStorage("sl-theme", "sun");
  const T = THEMES[theme];

  const [names, setNames] = useLocalStorage("sl-names", { a: "Sabrina", b: "Leiliane" });
  const [since, setSince] = useLocalStorage("sl-since", "");
  const [story, setStory] = useLocalStorage(
    "sl-story",
    "Nosso amor é encontro de Sol e Lua: luz e calma, calor e aconchego. Cada dia ao seu lado é poesia."
  );
  const [spotify, setSpotify] = useLocalStorage("sl-spotify", "");
  const [editing, setEditing] = useState(false);

  const [media, setMedia] = useState([]);
  const fileInputRef = useRef(null);

  const duration = useMemo(() => formatDuration(since), [since]);

  const handleFiles = (files) => {
    const items = [];
    Array.from(files).forEach((f) => {
      const url = URL.createObjectURL(f);
      const type = f.type.startsWith("video") ? "video" : "image";
      items.push({ id: `${Date.now()}-${Math.random()}`, type, url, name: f.name });
    });
    setMedia((m) => [...items, ...m]);
  };

  const removeItem = (id) => setMedia((m) => m.filter((x) => x.id !== id));

  const onDrop = (e) => {
    e.preventDefault();
    if (e.dataTransfer.files?.length) handleFiles(e.dataTransfer.files);
  };

  const onPaste = (e) => {
    const items = e.clipboardData?.items;
    if (!items) return;
    const files = [];
    for (let i = 0; i < items.length; i++) {
      const it = items[i];
      if (it.kind === "file") {
        const file = it.getAsFile();
        if (file) files.push(file);
      }
    }
    if (files.length) handleFiles(files);
  };

  const ThemeIcon = T.icon;

  return (
    <div
      className={`min-h-screen w-full bg-gradient-to-b ${T.bg} relative overflow-hidden`}
      onDragOver={(e) => e.preventDefault()}
      onDrop={onDrop}
      onPaste={onPaste}
    >
      <div className={`pointer-events-none absolute inset-0 ${T.glow}`} />
      {theme === "moon" && (
        <div className="pointer-events-none absolute inset-0">
          {Array.from({ length: 80 }).map((_, i) => (
            <div
              key={i}
              className="absolute rounded-full bg-white/70"
              style={{
                width: Math.random() * 2 + 1,
                height: Math.random() * 2 + 1,
                top: `${Math.random() * 100}%`,
                left: `${Math.random() * 100}%`,
                opacity: Math.random() * 0.8 + 0.2,
              }}
            />
          ))}
        </div>
      )}

      <header className="relative z-10 max-w-5xl mx-auto px-4 pt-8 pb-3 flex items-center justify-between">
        <motion.h1
          initial={{ opacity: 0, y: -10 }}
          animate={{ opacity: 1, y: 0 }}
          className={`font-bold text-2xl sm:text-3xl ${T.text} tracking-tight flex items-center gap-3`}
        >
          <span className="inline-flex h-10 w-10 items-center justify-center rounded-2xl shadow-inner border border-black/5 bg-white/60 backdrop-blur">
            <Heart className="h-5 w-5" />
          </span>
          Sabrina & Leiliane
          <Chip>Sol & Lua</Chip>
        </motion.h1>
        <div className="flex items-center gap-2">
          <Button className="bg-white/70" onClick={() => setEditing(true)}>
            <Settings className="h-4 w-4" /> Editar
          </Button>
          <Button className="bg-white/70" onClick={() => setTheme(theme === "sun" ? "moon" : "sun")}>
            <ThemeIcon className="h-4 w-4" /> {theme === "sun" ? "Lua" : "Sol"}
          </Button>
        </div>
      </header>

      <section className="relative z-10 max-w-5xl mx-auto px-4">
        <Card className="bg-white/70 backdrop-blur">
          <CardContent>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6 items-center">
              <div className="md:col-span-2">
                <motion.h2
                  initial={{ opacity: 0, y: 10 }}
                  animate={{ opacity: 1, y: 0 }}
                  className="text-3xl sm:text-4xl font-extrabold text-slate-800 flex items-center gap-3"
                >
                  {names.a} <Heart className="h-6 w-6" /> {names.b}
                </motion.h2>
                <p className="mt-2 text-slate-600 leading-relaxed">{story}</p>
                <div className="mt-4 flex flex-wrap items-center gap-3">
                  <div className="inline-flex items-center gap-2 rounded-2xl bg-white px-3 py-2 border border-black/5 shadow-sm">
                    <Calendar className="h-4 w-4" />
                    <span className="text-sm text-slate-700">Desde: {since ? new Date(since).toLocaleDateString() : "—"}</span>
                  </div>
                  <div className="inline-flex items-center gap-2 rounded-2xl bg-white px-3 py-2 border border-black/5 shadow-sm">
                    <Heart className="h-4 w-4" />
                    <span className="text-sm text-slate-700">{duration.text}</span>
                  </div>
                </div>
              </div>
              <div className="md:col-span-1">
                <div className="relative">
                  <div className="aspect-square rounded-3xl border border-black/5 shadow-inner p-6 bg-gradient-to-br from-white to-white/60 flex items-center justify-center">
                    <motion.div
                      initial={{ rotate: -10, opacity: 0 }}
                      animate={{ rotate: 0, opacity: 1 }}
                      transition={{ type: "spring", stiffness: 200, damping: 20 }}
                      className="relative"
                    >
                      <div className="relative h-28 w-28">
                        <div className="absolute inset-0 rounded-full bg-gradient-to-br from-amber-300 to-yellow-400 shadow" />
                        <div className="absolute inset-[10%] rounded-full bg-white/60 backdrop-blur border border-white/50" />
                        <div className="absolute right-2 top-2 h-9 w-9 rounded-full bg-gradient-to-br from-indigo-200 to-sky-200 shadow" />
                      </div>
                    </motion.div>
                  </div>
                </div>
              </div>
            </div>
          </CardContent>
        </Card>
      </section>

      <section className="relative z-10 max-w-5xl mx-auto px-4 mt-6">
        <div className="flex items-center justify-between mb-3">
          <h3 className="text-lg font-semibold text-slate-800 flex items-center gap-2"><ImageIcon className="h-5 w-5" /> Memórias</h3>
          <div className="flex items-center gap-2">
            <Button className="bg-white/70" onClick={() => fileInputRef.current?.click()}>
              <Upload className="h-4 w-4" /> Enviar fotos/vídeos
            </Button>
            <input ref={fileInputRef} type="file" accept="image/*,video/*" multiple className="hidden" onChange={(e) => e.target.files && handleFiles(e.target.files)} />
          </div>
        </div>

        <Card className="bg-white/70 backdrop-blur">
          <CardContent>
            {media.length === 0 ? (
              <div className="text-center py-14 border-2 border-dashed rounded-3xl border-black/10" onClick={() => fileInputRef.current?.click()}>
                <p className="text-slate-600 max-w-md mx-auto">
                  Arraste e solte arquivos aqui, clique para escolher, ou cole capturas (Ctrl/Cmd+V).<br />
                  <span className="text-xs text-slate-500">As mídias ficam somente nesta sessão.</span>
                </p>
              </div>
            ) : (
              <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-3">
                {media.map((item) => (
                  <div key={item.id} className="group relative">
                    {item.type === "image" ? (
                      <img src={item.url} alt={item.name} className="h-40 w-full object-cover rounded-2xl border border-black/5 shadow" />
                    ) : (
                      <video src={item.url} controls className="h-40 w-full object-cover rounded-2xl border border-black/5 shadow" />
                    )}
                    <div className="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition">
                      <Button className="bg-white/90 px-2 py-1" onClick={() => removeItem(item.id)}>
                        <Trash2 className="h-4 w-4" />
                      </Button>
                    </div>
                    <div className="absolute bottom-2 left-2">
                      <Chip className="bg-white/80">{item.type === "image" ? <ImageIcon className="inline h-3 w-3" /> : <VideoIcon className="inline h-3 w-3" />} <span className="ml-1 text-xs">{item.type === "image" ? "Foto" : "Vídeo"}</span></Chip>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </CardContent>
        </Card>
      </section>

      {/* Spotify Playlist */}
      {spotify && (
        <section className="relative z-10 max-w-5xl mx-auto px-4 mt-6">
          <h3 className="text-lg font-semibold text-slate-800 flex items-center gap-2 mb-3"><Music className="h-5 w-5" /> Nossa Trilha Sonora</h3>
          <Card className="bg-white/70 backdrop-blur">
            <CardContent>
              <div className="aspect-video w-full">
                <iframe
                  src={`https://open.spotify.com/embed/playlist/${spotify}`}
                  width="100%"
                  height="100%"
                  frameBorder="0"
                  allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                  loading="lazy"
                  className="rounded-2xl"
                ></iframe>
              </div>
            </CardContent>
          </Card>
        </section>
      )}

      <footer className="relative z-10 max-w-5xl mx-auto px-4 py-10 text-center">
        <p className="text-sm text-slate-600">Feito com muito ❤️ por {names.a} & {names.b}. Tema {T.name}.</p>
      </footer>

      {editing && (
        <div className="fixed inset-0 z-50 grid place-items-end sm:place-items-center">
          <div className="absolute inset-0 bg-black/40" onClick={() => setEditing(false)} />
          <motion.div
            initial={{ y: 40, opacity: 0 }}
            animate={{ y: 0, opacity: 1 }}
            className="relative w-full sm:w-[560px] bg-white rounded-t-3xl sm:rounded-3xl shadow-xl border border-black/10"
          >
            <div className="p-5 border-b border-black/5 flex items-center justify-between">
              <div className="flex items-center
