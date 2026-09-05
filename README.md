# API_google_maps
Scopo
git commit
import React, { useState, useEffect, useRef } from 'react';
import { 
  Search, Map as MapIcon, List, PlusCircle, 
  Heart, Navigation, X, Star, Info, Filter,
  Phone, Globe, Clock, ShieldCheck
} from 'lucide-react';
import { Loader } from '@googlemaps/js-api-loader';

// --- Interfaces ---
interface Place {
  id: string;
  name: string;
  address: string;
  lat: number;
  lng: number;
  type: string;
  isPublic: boolean;
  isFree: boolean;
  rating: number;
  reviews: number;
  image: string;
}

// --- Mock Data ---
const GYN_PLACES: Place[] = [
  { id: '1', name: 'Campo do Setor Bueno', address: 'Praça T-25, Setor Bueno, Goiânia', lat: -16.7025, lng: -49.2652, type: 'campo_futebol', isPublic: true, isFree: true, rating: 4.5, reviews: 120, image: 'https://images.unsplash.com/photo-1529900748604-07564a03e7a6?w=400' },
  { id: '2', name: 'Arena Society Marista', address: 'Rua 145, Setor Marista, Goiânia', lat: -16.6990, lng: -49.2590, type: 'campo_society', isPublic: false, isFree: false, rating: 4.8, reviews: 85, image: 'https://images.unsplash.com/photo-1574629810360-7efbbe195018?w=400' },
  { id: '3', name: 'Ginásio Rio Vermelho', address: 'Av. Paranaíba, Centro, Goiânia', lat: -16.6715, lng: -49.2625, type: 'ginasio', isPublic: true, isFree: true, rating: 4.2, reviews: 210, image: 'https://images.unsplash.com/photo-1504450758481-7338eba7524a?w=400' },
];

export default function App() {
  const [view, setView] = useState<'map' | 'list'>('map');
  const [selectedPlace, setSelectedPlace] = useState<Place | null>(null);
  const [isSuggesting, setIsSuggesting] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const mapRef = useRef<HTMLDivElement>(null);

  // Inicializa o Google Maps
  useEffect(() => {
    if (mapRef.current && view === 'map') {
      const loader = new Loader({
        apiKey: import.meta.env.VITE_GOOGLE_MAPS_KEY || "",
        version: "weekly",
        libraries: ["places"]
      });

      loader.load().then((google) => {
        const map = new google.maps.Map(mapRef.current!, {
          center: { lat: -16.6869, lng: -49.2648 },
          zoom: 14,
          disableDefaultUI: true,
          styles: [{ featureType: "poi", elementType: "labels", stylers: [{ visibility: "off" }] }]
        });

        GYN_PLACES.forEach(place => {
          const marker = new google.maps.Marker({
            position: { lat: place.lat, lng: place.lng },
            map,
            icon: {
              path: google.maps.SymbolPath.CIRCLE,
              scale: 10,
              fillColor: "#15803d",
              fillOpacity: 1,
              strokeWeight: 2,
              strokeColor: "#ffffff",
            }
          });
          marker.addListener('click', () => setSelectedPlace(place));
        });
      });
    }
  }, [view]);

  return (
    <div className="flex flex-col h-screen w-full bg-white dark:bg-slate-950 text-slate-900 dark:text-white overflow-hidden">
      
      {/* HEADER (Safe Area for iPhone Notch) */}
      <header className="pt-[env(safe-area-inset-top)] bg-green-700 text-white z-30 shadow-md">
        <div className="px-5 h-16 flex items-center gap-3">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-green-200" size={18} />
            <input 
              type="text"
              placeholder="Buscar em Goiânia..."
              className="w-full bg-green-800/50 border-none rounded-2xl py-2.5 pl-10 pr-4 text-sm placeholder:text-green-200 focus:ring-2 focus:ring-white transition-all"
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
            />
          </div>
          <button className="p-2.5 bg-green-600 rounded-2xl active:scale-90 transition-transform">
            <Filter size={20} />
          </button>
        </div>
      </header>

      {/* MAIN CONTENT */}
      <main className="flex-1 relative overflow-hidden">
        {view === 'map' ? (
          <div ref={mapRef} className="w-full h-full" />
        ) : (
          <div className="h-full overflow-y-auto p-5 space-y-4 pb-32">
            {GYN_PLACES.map(place => (
              <div 
                key={place.id} 
                onClick={() => { setSelectedPlace(place); setView('map'); }}
                className="flex gap-4 bg-slate-50 dark:bg-slate-900 p-3 rounded-3xl border border-slate-100 dark:border-slate-800 active:bg-slate-100 transition-colors"
              >
                <img src={place.image} className="w-24 h-24 rounded-2xl object-cover" alt={place.name} />
                <div className="flex-1 py-1">
                  <h3 className="font-bold text-base leading-tight mb-1">{place.name}</h3>
                  <p className="text-xs text-slate-500 line-clamp-1 mb-2">{place.address}</p>
                  <div className="flex items-center gap-2">
                    <span className="flex items-center text-yellow-500 font-bold text-xs"><Star size={12} className="fill-current mr-0.5"/> {place.rating}</span>
                    <span className="text-[10px] px-2 py-0.5 rounded-md bg-green-100 text-green-700 font-bold uppercase">{place.isFree ? 'Grátis' : 'Pago'}</span>
                  </div>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* BOTTOM SHEET (iOS Style) */}
        {selectedPlace && (
          <div className="absolute inset-0 z-40 bg-black/20" onClick={() => setSelectedPlace(null)}>
            <div 
              className="absolute inset-x-0 bottom-0 bg-white dark:bg-slate-900 rounded-t-[40px] shadow-2xl p-8 pb-[env(safe-area-inset-bottom)] animate-in slide-in-from-bottom duration-300"
              onClick={e => e.stopPropagation()}
            >
              <div className="w-12 h-1.5 bg-slate-200 dark:bg-slate-700 rounded-full mx-auto mb-6" />
              
              <div className="flex justify-between items-start mb-6">
                <div className="flex-1 pr-4">
                  <h2 className="text-2xl font-black mb-1">{selectedPlace.name}</h2>
                  <p className="text-slate-500 text-sm flex items-center gap-1"><MapIcon size={14}/> {selectedPlace.address}</p>
                </div>
                <button onClick={() => setSelectedPlace(null)} className="p-2 bg-slate-100 dark:bg-slate-800 rounded-full"><X size={20} /></button>
              </div>

              <div className="grid grid-cols-2 gap-4 mb-8">
                <div className="flex items-center gap-3 p-4 bg-slate-50 dark:bg-slate-800 rounded-2xl">
                  <ShieldCheck className="text-green-600" />
                  <div><p className="text-[10px] uppercase font-bold text-slate-400">Tipo</p><p className="text-sm font-bold">Público</p></div>
                </div>
                <div className="flex items-center gap-3 p-4 bg-slate-50 dark:bg-slate-800 rounded-2xl">
                  <Clock className="text-blue-600" />
                  <div><p className="text-[10px] uppercase font-bold text-slate-400">Status</p><p className="text-sm font-bold text-green-600">Aberto</p></div>
                </div>
              </div>

              <div className="flex gap-3">
                <button className="flex-1 bg-green-700 text-white py-4 rounded-2xl font-bold flex items-center justify-center gap-2 active:scale-95 transition-transform shadow-lg shadow-green-200 dark:shadow-none">
                  <Navigation size={20} /> Traçar Rota
                </button>
                <button className="p-4 bg-slate-100 dark:bg-slate-800 rounded-2xl active:scale-95 transition-transform"><Heart size={24} className="text-slate-400" /></button>
              </div>
            </div>
          </div>
        )}
      </main>

      {/* TAB BAR (iPhone 13 Pro Max style) */}
      <nav className="bg-white/90 dark:bg-slate-900/90 backdrop-blur-xl border-t border-slate-100 dark:border-slate-800 pb-[env(safe-area-inset-bottom)] z-50">
        <div className="flex justify-around items-center h-20">
          <button onClick={() => setView('map')} className={`flex flex-col items-center gap-1 ${view === 'map' ? 'text-green-700' : 'text-slate-400'}`}>
            <MapIcon size={24} strokeWidth={view === 'map' ? 2.5 : 2} />
            <span className="text-[10px] font-black uppercase tracking-widest">Mapa</span>
          </button>
          <button onClick={() => setIsSuggesting(true)} className="flex flex-col items-center -mt-10">
            <div className="bg-green-700 p-4 rounded-full shadow-xl shadow-green-300 dark:shadow-none border-4 border-white dark:border-slate-950 active:scale-90 transition-transform">
              <PlusCircle size={28} className="text-white" />
            </div>
            <span className="text-[10px] font-black uppercase tracking-widest mt-2 text-slate-400">Sugerir</span>
          </button>
          <button onClick={() => setView('list')} className={`flex flex-col items-center gap-1 ${view === 'list' ? 'text-green-700' : 'text-slate-400'}`}>
            <List size={24} strokeWidth={view === 'list' ? 2.5 : 2} />
            <span className="text-[10px] font-black uppercase tracking-widest">Lista</span>
          </button>
        </div>
      </nav>

      {/* SUGGESTION MODAL */}
      {isSuggesting && (
        <div className="fixed inset-0 z-[60] bg-black/60 backdrop-blur-sm flex items-end">
          <div className="bg-white dark:bg-slate-950 w-full h-[92vh] rounded-t-[40px] flex flex-col p-8 animate-in slide-in-from-bottom duration-500">
            <div className="flex justify-between items-center mb-8">
              <h2 className="text-2xl font-black">Novo Local</h2>
              <button onClick={() => setIsSuggesting(false)} className="p-2 bg-slate-100 dark:bg-slate-800 rounded-full"><X /></button>
            </div>
            <div className="flex-1 overflow-y-auto space-y-6">
              <div className="space-y-2">
                <label className="text-xs font-black uppercase text-slate-400 ml-1">Nome do Campo/Quadra</label>
                <input className="w-full p-4 bg-slate-50 dark:bg-slate-900 rounded-2xl border-none focus:ring-2 focus:ring-green-600 transition-all" placeholder="Ex: Arena Gyn Society" />
              </div>
              <div className="space-y-2">
                <label className="text-xs font-black uppercase text-slate-400 ml-1">Endereço ou Bairro</label>
                <input className="w-full p-4 bg-slate-50 dark:bg-slate-900 rounded-2xl border-none focus:ring-2 focus:ring-green-600 transition-all" placeholder="Ex: Setor Sul" />
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div className="space-y-2">
                  <label className="text-xs font-black uppercase text-slate-400 ml-1">Tipo de Acesso</label>
                  <select className="w-full p-4 bg-slate-50 dark:bg-slate-900 rounded-2xl border-none font-bold">
                    <option>Público</option>
                    <option>Privado</option>
                  </select>
                </div>
                <div className="space-y-2">
                  <label className="text-xs font-black uppercase text-slate-400 ml-1">Custo</label>
                  <select className="w-full p-4 bg-slate-50 dark:bg-slate-900 rounded-2xl border-none font-bold">
                    <option>Gratuito</option>
                    <option>Pago / Aluguel</option>
                  </select>
                </div>
              </div>
              <button 
                onClick={() => setIsSuggesting(false)}
                className="w-full bg-green-700 text-white py-5 rounded-3xl font-black text-lg shadow-xl active:scale-95 transition-transform"
              >
                ENVIAR PARA REVISÃO
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
git public
