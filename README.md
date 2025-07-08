<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistema de Controle de Competição de Natação</title>
  <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.22.9/Babel.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/lucide-react@0.263.0/dist/umd/lucide-react.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    const { useState, useEffect } = React;
    const { Plus, Users, Trophy, Clock, FileText, Medal, Search, Edit, Trash2, Flag, BarChart2, CheckCircle, AlertCircle } = LucideReact;

    const SwimmingCompetitionSystem = () => {
      const [activeTab, setActiveTab] = useState('dashboard');
      const [athletes, setAthletes] = useState([]);
      const [events, setEvents] = useState([]);
      const [registrations, setRegistrations] = useState([]);
      const [results, setResults] = useState([]);
      const [trainingData, setTrainingData] = useState([]);
      const [infractions, setInfractions] = useState([]);
      const [showModal, setShowModal] = useState(false);
      const [modalType, setModalType] = useState('');
      const [editingItem, setEditingItem] = useState(null);
      const [currentEventControl, setCurrentEventControl] = useState(null);
      const [checkedAthletes, setCheckedAthletes] = useState([]);

      const [athleteForm, setAthleteForm] = useState({
        id: 0, name: '', age: '', club: '', category: '', gender: ''
      });
      const [eventForm, setEventForm] = useState({
        id: 0, name: '', distance: '', stroke: '', category: '', gender: '', datetime: '', maxParticipants: 8
      });
      const [resultForm, setResultForm] = useState({
        id: 0, athleteId: 0, eventId: 0, time: '', lane: '', position: '', points: 0
      });
      const [trainingForm, setTrainingForm] = useState({
        id: 0, athleteId: 0, date: '', volume: 0, intensity: 0, notes: ''
      });
      const [infractionForm, setInfractionForm] = useState({
        id: 0, athleteId: 0, eventId: 0, type: '', description: '', penalty: ''
      });

      useEffect(() => {
        setAthletes([
          { id: 1, name: 'Leonardo Pontes', age: '15', club: 'Clube de Engenharia', category: 'Juvenil', gender: 'M' },
          { id: 2, name: 'Laysa Pontes', age: '18', club: 'Athletica UFG', category: 'Junior', gender: 'F' },
          { id: 3, name: 'Marcelo Pontes', age: '35', club: 'AGMN', category: 'Senior', gender: 'M' }
        ]);
        setEvents([
          { id: 1, name: '100m Peito Masculino', distance: '100m', stroke: 'Peito', category: 'Juvenil', gender: 'M', datetime: '2025-08-08T08:08', maxParticipants: 8 },
          { id: 2, name: '50m Costas Feminino', distance: '50m', stroke: 'Costas', category: 'Junior', gender: 'F', datetime: '2025-08-08T08:08', maxParticipants: 8 }
        ]);
        setRegistrations([
          { id: 1, athleteId: 1, eventId: 1, lane: 4, status: 'Confirmado' },
          { id: 2, athleteId: 2, eventId: 2, lane: 3, status: 'Confirmado' }
        ]);
        setResults([
          { id: 1, athleteId: 1, eventId: 1, time: '1:02.45', lane: '4', position: '1', points: 20 },
          { id: 2, athleteId: 2, eventId: 2, time: '27.89', lane: '3', position: '1', points: 20 }
        ]);
        setTrainingData([
          { id: 1, athleteId: 1, date: '2025-08-08', volume: 5000, intensity: 7, notes: 'Bom desempenho' },
          { id: 2, athleteId: 2, date: '2025-08-08', volume: 4500, intensity: 8, notes: 'Cansado' }
        ]);
        setInfractions([
          { id: 1, athleteId: 1, eventId: 1, type: 'Saída falsa', description: 'Saída antes do sinal', penalty: 'Desclassificação' }
        ]);
      }, []);

      const getAthleteName = (id) => athletes.find(a => a.id === id)?.name || 'N/A';
      const getEventName = (id) => events.find(e => e.id === id)?.name || 'N/A';

      const handleSubmit = (type) => {
        const id = editingItem ? editingItem.id : Date.now();
        if (type === 'athlete') {
          if (editingItem) {
            setAthletes(athletes.map(a => a.id === editingItem.id ? { ...athleteForm, id } : a));
          } else {
            setAthletes([...athletes, { ...athleteForm, id }]);
          }
          setAthleteForm({ id: 0, name: '', age: '', club: '', category: '', gender: '' });
        } else if (type === 'event') {
          if (editingItem) {
            setEvents(events.map(e => e.id === editingItem.id ? { ...eventForm, id } : e));
          } else {
            setEvents([...events, { ...eventForm, id }]);
          }
          setEventForm({ id: 0, name: '', distance: '', stroke: '', category: '', gender: '', datetime: '', maxParticipants: 8 });
        } else if (type === 'result') {
          const newResult = { ...resultForm, id, points: calculatePoints(resultForm.position) };
          if (editingItem) {
            setResults(results.map(r => r.id === editingItem.id ? newResult : r));
          } else {
            setResults([...results, newResult]);
          }
          setResultForm({ id: 0, athleteId: 0, eventId: 0, time: '', lane: '', position: '', points: 0 });
        } else if (type === 'training') {
          if (editingItem) {
            setTrainingData(trainingData.map(t => t.id === editingItem.id ? { ...trainingForm, id } : t));
          } else {
            setTrainingData([...trainingData, { ...trainingForm, id }]);
          }
          setTrainingForm({ id: 0, athleteId: 0, date: '', volume: 0, intensity: 0, notes: '' });
        } else if (type === 'infraction') {
          if (editingItem) {
            setInfractions(infractions.map(i => i.id === editingItem.id ? { ...infractionForm, id } : i));
          } else {
            setInfractions([...infractions, { ...infractionForm, id }]);
          }
          setInfractionForm({ id: 0, athleteId: 0, eventId: 0, type: '', description: '', penalty: '' });
        }
        setShowModal(false);
        setEditingItem(null);
      };

      const calculatePoints = (position) => {
        const pointsTable = { '1': 20, '2': 17, '3': 16, '4': 15, '5': 14, '6': 13, '7': 12, '8': 11 };
        return pointsTable[position] || 0;
      };

      const openModal = (type, item = null) => {
        setModalType(type);
        setEditingItem(item);
        if (item) {
          if (type === 'athlete') setAthleteForm(item);
          else if (type === 'event') setEventForm(item);
          else if (type === 'result') setResultForm(item);
          else if (type === 'training') setTrainingForm(item);
          else if (type === 'infraction') setInfractionForm(item);
        }
        setShowModal(true);
      };

      const deleteItem = (type, id) => {
        if (type === 'athlete') setAthletes(athletes.filter(a => a.id !== id));
        else if (type === 'event') setEvents(events.filter(e => e.id !== id));
        else if (type === 'result') setResults(results.filter(r => r.id !== id));
        else if (type === 'training') setTrainingData(trainingData.filter(t => t.id !== id));
        else if (type === 'infraction') setInfractions(infractions.filter(i => i.id !== id));
      };

      const getAthleteRanking = () => {
        return athletes
          .map(athlete => ({
            ...athlete,
            points: results
              .filter(r => r.athleteId === athlete.id)
              .reduce((sum, r) => sum + r.points, 0)
          }))
          .sort((a, b) => b.points - a.points);
      };

      const generateClubReport = () => {
        const clubs = [...new Set(athletes.map(a => a.club))];
        return clubs.map(club => {
          const clubAthletes = athletes.filter(a => a.club === club);
          const clubPoints = clubAthletes.reduce((sum, athlete) => {
            return sum + results.filter(r => r.athleteId === athlete.id).reduce((s, r) => s + r.points, 0);
          }, 0);
          return {
            club,
            athletes: clubAthletes.length,
            points: clubPoints,
            gold: results.filter(r => r.position === '1' && clubAthletes.some(a => a.id === r.athleteId)).length
          };
        }).sort((a, b) => b.points - a.points);
      };

      const handleCheckIn = (athleteId) => {
        if (!checkedAthletes.includes(athleteId)) {
          setCheckedAthletes([...checkedAthletes, athleteId]);
        }
      };

      const Modal = () => (
        showModal && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white rounded-lg p-6 w-full max-w-md mx-4">
              <h3 className="text-lg font-bold mb-4">
                {editingItem ? 'Editar' : 'Adicionar'} {modalType === 'athlete' ? 'Atleta' : modalType === 'event' ? 'Prova' : modalType === 'result' ? 'Resultado' : modalType === 'training' ? 'Treino' : 'Infração'}
              </h3>
              {modalType === 'athlete' && (
                <div className="space-y-4">
                  <input
                    type="text"
                    placeholder="Nome do atleta"
                    value={athleteForm.name}
                    onChange={(e) => setAthleteForm({ ...athleteForm, name: e.target.value })}
                    className="w-full p-2 border rounded"
 precisamos implementar as seções que estavam faltando no código original, como o **Dashboard**, **Athletes**, **Events**, **Registrations** e **Results**, além de garantir que o código seja executável diretamente no navegador. Aqui está a continuação do componente `SwimmingCompetitionSystem` com todas as seções implementadas:

```jsx
                  />
                  <input
                    type="number"
                    placeholder="Idade"
                    value={athleteForm.age}
                    onChange={(e) => setAthleteForm({ ...athleteForm, age: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <input
                    type="text"
                    placeholder="Clube"
                    value={athleteForm.club}
                    onChange={(e) => setAthleteForm({ ...athleteForm, club: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <select
                    value={athleteForm.category}
                    onChange={(e) => setAthleteForm({ ...athleteForm, category: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a categoria</option>
                    <option value="Mirim">Mirim (9-10)</option>
                    <option value="Petiz">Petiz (11-12)</option>
                    <option value="Infantil">Infantil (13-14)</option>
                    <option value="Juvenil">Juvenil (15-17)</option>
                    <option value="Junior">Junior (18-19)</option>
                    <option value="Senior">Senior (20+)</option>
                  </select>
                  <select
                    value={athleteForm.gender}
                    onChange={(e) => setAthleteForm({ ...athleteForm, gender: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o gênero</option>
                    <option value="M">Masculino</option>
                    <option value="F">Feminino</option>
                  </select>
                </div>
              )}
              {modalType === 'event' && (
                <div className="space-y-4">
                  <input
                    type="text"
                    placeholder="Nome da prova"
                    value={eventForm.name}
                    onChange={(e) => setEventForm({ ...eventForm, name: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <select
                    value={eventForm.distance}
                    onChange={(e) => setEventForm({ ...eventForm, distance: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a distância</option>
                    <option value="25m">25m</option>
                    <option value="50m">50m</option>
                    <option value="100m">100m</option>
                    <option value="200m">200m</option>
                    <option value="400m">400m</option>
                    <option value="800m">800m</option>
                    <option value="1500m">1500m</option>
                  </select>
                  <select
                    value={eventForm.stroke}
                    onChange={(e) => setEventForm({ ...eventForm, stroke: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o nado</option>
                    <option value="Livre">Livre</option>
                    <option value="Costas">Costas</option>
                    <option value="Peito">Peito</option>
                    <option value="Borboleta">Borboleta</option>
                    <option value="Medley">Medley</option>
                  </select>
                  <select
                    value={eventForm.category}
                    onChange={(e) => setEventForm({ ...eventForm, category: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a categoria</option>
                    <option value="Mirim">Mirim</option>
                    <option value="Petiz">Petiz</option>
                    <option value="Infantil">Infantil</option>
                    <option value="Juvenil">Juvenil</option>
                    <option value="Junior">Junior</option>
                    <option value="Senior">Senior</option>
                  </select>
                  <select
                    value={eventForm.gender}
                    onChange={(e) => setEventForm({ ...eventForm, gender: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o gênero</option>
                    <option value="M">Masculino</option>
                    <option value="F">Feminino</option>
                  </select>
                  <input
                    type="datetime-local"
                    value={eventForm.datetime}
                    onChange={(e) => setEventForm({ ...eventForm, datetime: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <input
                    type="number"
                    placeholder="Máximo de participantes"
                    value={eventForm.maxParticipants}
                    onChange={(e) => setEventForm({ ...eventForm, maxParticipants: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                </div>
              )}
              {modalType === 'result' && (
                <div className="space-y-4">
                  <select
                    value={resultForm.athleteId}
                    onChange={(e) => setResultForm({ ...resultForm, athleteId: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o atleta</option>
                    {athletes.map(athlete => (
                      <option key={athlete.id} value={athlete.id}>{athlete.name}</option>
                    ))}
                  </select>
                  <select
                    value={resultForm.eventId}
                    onChange={(e) => setResultForm({ ...resultForm, eventId: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a prova</option>
                    {events.map(event => (
                      <option key={event.id} value={event.id}>{event.name}</option>
                    ))}
                  </select>
                  <input
                    type="text"
                    placeholder="Tempo (ex: 1:02.45)"
                    value={resultForm.time}
                    onChange={(e) => setResultForm({ ...resultForm, time: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <input
                    type="number"
                    placeholder="Raia"
                    value={resultForm.lane}
                    onChange={(e) => setResultForm({ ...resultForm, lane: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <input
                    type="number"
                    placeholder="Posição"
                    value={resultForm.position}
                    onChange={(e) => setResultForm({ ...resultForm, position: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                </div>
              )}
              {modalType === 'training' && (
                <div className="space-y-4">
                  <select
                    value={trainingForm.athleteId}
                    onChange={(e) => setTrainingForm({ ...trainingForm, athleteId: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o atleta</option>
                    {athletes.map(athlete => (
                      <option key={athlete.id} value={athlete.id}>{athlete.name}</option>
                    ))}
                  </select>
                  <input
                    type="date"
                    value={trainingForm.date}
                    onChange={(e) => setTrainingForm({ ...trainingForm, date: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <input
                    type="number"
                    placeholder="Volume (metros)"
                    value={trainingForm.volume}
                    onChange={(e) => setTrainingForm({ ...trainingForm, volume: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <select
                    value={trainingForm.intensity}
                    onChange={(e) => setTrainingForm({ ...trainingForm, intensity: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a intensidade (1-10)</option>
                    {[1, 2, 3, 4, 5, 6, 7, 8, 9, 10].map(n => (
                      <option key={n} value={n}>{n}</option>
                    ))}
                  </select>
                  <textarea
                    placeholder="Observações"
                    value={trainingForm.notes}
                    onChange={(e) => setTrainingForm({ ...trainingForm, notes: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                </div>
              )}
              {modalType === 'infraction' && (
                <div className="space-y-4">
                  <select
                    value={infractionForm.athleteId}
                    onChange={(e) => setInfractionForm({ ...infractionForm, athleteId: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o atleta</option>
                    {athletes.map(athlete => (
                      <option key={athlete.id} value={athlete.id}>{athlete.name}</option>
                    ))}
                  </select>
                  <select
                    value={infractionForm.eventId}
                    onChange={(e) => setInfractionForm({ ...infractionForm, eventId: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a prova</option>
                    {events.map(event => (
                      <option key={event.id} value={event.id}>{event.name}</option>
                    ))}
                  </select>
                  <select
                    value={infractionForm.type}
                    onChange={(e) => setInfractionForm({ ...infractionForm, type: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione o tipo de infração</option>
                    <option value="Saída falsa">Saída falsa</option>
                    <option value="Nado irregular">Nado irregular</option>
                    <option value="Virada irregular">Virada irregular</option>
                    <option value="Outra">Outra</option>
                  </select>
                  <textarea
                    placeholder="Descrição"
                    value={infractionForm.description}
                    onChange={(e) => setInfractionForm({ ...infractionForm, description: e.target.value })}
                    className="w-full p-2 border rounded"
                  />
                  <select
                    value={infractionForm.penalty}
                    onChange={(e) => setInfractionForm({ ...infractionForm, penalty: e.target.value })}
                    className="w-full p-2 border rounded"
                  >
                    <option value="">Selecione a penalidade</option>
                    <option value="Advertência">Advertência</option>
                    <option value="Desclassificação">Desclassificação</option>
                    <option value="Suspensão">Suspensão</option>
                  </select>
                </div>
              )}
              <div className="flex justify-end space-x-2 mt-6">
                <button
                  onClick={() => { setShowModal(false); setEditingItem(null); }}
                  className="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
                >
                  Cancelar
                </button>
                <button
                  onClick={() => handleSubmit(modalType)}
                  className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
                >
                  {editingItem ? 'Salvar' : 'Adicionar'}
                </button>
              </div>
            </div>
          </div>
        )
      );

      const PerformanceChart = ({ athleteId }) => {
        const athleteTrainings = trainingData.filter(t => t.athleteId === athleteId);
        return (
          <div className="p-4 bg-white rounded-lg shadow">
            <h3 className="font-bold mb-2">Evolução de Desempenho</h3>
            <div className="h-40 flex items-end space-x-1">
              {athleteTrainings.map((training, index) => (
                <div key={index} className="flex-1 flex flex-col items-center">
                  <div
                    className="w-full bg-blue-500 rounded-t"
                    style={{ height: `${training.intensity * 10}%` }}
                  ></div>
                  <span className="text-xs mt-1">{training.date.split('-')[2]}</span>
                </div>
              ))}
            </div>
            <div className="mt-2 text-sm text-gray-600">
              <span className="font-medium">Último volume:</span> {athleteTrainings[0]?.volume || 0}m
            </div>
          </div>
        );
      };

      const DashboardTab = () => (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
          <div className="bg-white p-6 rounded-lg shadow">
            <div className="flex items-center">
              <Users className="w-8 h-8 text-blue-600 mr-4" />
              <div>
                <h3 className="text-lg font-semibold">Atletas</h3>
                <p className="text-2xl font-bold">{athletes.length}</p>
              </div>
            </div>
          </div>
          <div className="bg-white p-6 rounded-lg shadow">
            <div className="flex items-center">
              <Flag className="w-8 h-8 text-blue-600 mr-4" />
              <div>
                <h3 className="text-lg font-semibold">Provas</h3>
                <p className="text-2xl font-bold">{events.length}</p>
              </div>
            </div>
          </div>
          <div className="bg-white p-6 rounded-lg shadow">
            <div className="flex items-center">
              <Medal className="w-8 h-8 text-blue-600 mr-4" />
              <div>
                <h3 className="text-lg font-semibold">Resultados</h3>
                <p className="text-2xl font-bold">{results.length}</p>
              </div>
            </div>
          </div>
          <div className="bg-white p-6 rounded-lg shadow">
            <div className="flex items-center">
              <AlertCircle className="w-8 h-8 text-blue-600 mr-4" />
              <div>
                <h3 className="text-lg font-semibold">Infrações</h3>
                <p className="text-2xl font-bold">{infractions.length}</p>
              </div>
            </div>
          </div>
        </div>
      );

      const AthletesTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Atletas</h2>
              <button
                onClick={() => openModal('athlete')}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 flex items-center"
              >
                <Plus className="w-4 h-4 mr-2" />
                Novo Atleta
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Nome</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Idade</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Clube</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Categoria</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Gênero</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {athletes.map(athlete => (
                  <tr key={athlete.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{athlete.name}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{athlete.age}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{athlete.club}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{athlete.category}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{athlete.gender === 'M' ? 'Masculino' : 'Feminino'}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('athlete', athlete)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('athlete', athlete.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const EventsTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Provas</h2>
              <button
                onClick={() => openModal('event')}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 flex items-center"
              >
                <Plus className="w-4 h-4 mr-2" />
                Nova Prova
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Nome</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Distância</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Nado</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Categoria</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Gênero</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Data/Hora</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {events.map(event => (
                  <tr key={event.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{event.name}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{event.distance}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{event.stroke}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{event.category}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{event.gender === 'M' ? 'Masculino' : 'Feminino'}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{event.datetime}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('event', event)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('event', event.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const RegistrationsTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Inscrições</h2>
              <button
                onClick={() => openModal('registration')}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 flex items-center"
              >
                <Plus className="w-4 h-4 mr-2" />
                Nova Inscrição
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Atleta</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Prova</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Raia</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Status</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {registrations.map(reg => (
                  <tr key={reg.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{getAthleteName(reg.athleteId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{getEventName(reg.eventId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{reg.lane}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{reg.status}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('registration', reg)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('registration', reg.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const ResultsTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Resultados</h2>
              <button
                onClick={() => openModal('result')}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 flex items-center"
              >
                <Plus className="w-4 h-4 mr-2" />
                Novo Resultado
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Atleta</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Prova</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Tempo</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Raia</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Posição</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Pontos</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {results.map(result => (
                  <tr key={result.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{getAthleteName(result.athleteId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{getEventName(result.eventId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{result.time}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{result.lane}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{result.position}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{result.points}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('result', result)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('result', result.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const TrainingTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Controle de Treinos</h2>
              <button
                onClick={() => openModal('training')}
                className="bg-purple-600 text-white px-4 py-2 rounded-lg hover:bg-purple-700 flex items-center"
              >
                <Plus className="w-4 h-4 mr-2" />
                Novo Registro
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Atleta</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Data</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Volume (m)</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Intensidade</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Observações</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {trainingData.map(training => (
                  <tr key={training.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{getAthleteName(training.athleteId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{training.date}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{training.volume}m</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex items-center">
                        <div className="w-16 bg-gray-200 rounded-full h-2.5">
                          <div
                            className="bg-blue-600 h-2.5 rounded-full"
                            style={{ width: `${training.intensity * 10}%` }}
                          ></div>
                        </div>
                        <span className="ml-2 text-sm">{training.intensity}/10</span>
                      </div>
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap max-w-xs truncate">{training.notes}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('training', training)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('training', training.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const ControlDesk = () => (
        <div className="bg-white rounded-lg shadow-md p-6">
          <h2 className="text-xl font-bold mb-4">Banco de Controle</h2>
          <select
            onChange={(e) => setCurrentEventControl(events.find(ev => ev.id === parseInt(e.target.value)))}
            className="w-full p-2 border rounded mb-4"
          >
            <option value="">Selecione a prova</option>
            {events.map(event => (
              <option key={event.id} value={event.id}>{event.name}</option>
            ))}
          </select>
          {currentEventControl && (
            <div>
              <h3 className="font-medium mb-2">Atletas inscritos:</h3>
              <ul className="space-y-2">
                {registrations
                  .filter(reg => reg.eventId === currentEventControl.id)
                  .map(reg => (
                    <li key={reg.id} className="flex items-center justify-between p-2 border rounded">
                      <span>{getAthleteName(reg.athleteId)}</span>
                      <button
                        onClick={() => handleCheckIn(reg.athleteId)}
                        disabled={checkedAthletes.includes(reg.athleteId)}
                        className={`px-3 py-1 rounded ${
                          checkedAthletes.includes(reg.athleteId)
                            ? 'bg-green-100 text-green-800'
                            : 'bg-blue-100 text-blue-800 hover:bg-blue-200'
                        }`}
                      >
                        {checkedAthletes.includes(reg.athleteId) ? 'Presente' : 'Confirmar'}
                      </button>
                    </li>
                  ))}
              </ul>
            </div>
          )}
        </div>
      );

      const JudiciaryTab = () => (
        <div className="bg-white rounded-lg shadow-md">
          <div className="p-6 border-b border-gray-200">
            <div className="flex justify-between items-center">
              <h2 className="text-xl font-bold">Judiciária</h2>
              <button
                onClick={() => openModal('infraction')}
                className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 flex items-center"
              >
                <AlertCircle className="w-4 h-4 mr-2" />
                Registrar Infração
              </button>
            </div>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="bg-gray-50">
                <tr>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Atleta</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Prova</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Tipo</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Descrição</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Penalidade</th>
                  <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Ações</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-200">
                {infractions.map(infraction => (
                  <tr key={infraction.id}>
                    <td className="px-6 py-4 whitespace-nowrap">{getAthleteName(infraction.athleteId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">{getEventName(infraction.eventId)}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <span className="px-2 py-1 bg-red-100 text-red-800 rounded-full text-sm">
                        {infraction.type}
                      </span>
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap">{infraction.description}</td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <span className="px-2 py-1 bg-orange-100 text-orange-800 rounded-full text-sm">
                        {infraction.penalty}
                      </span>
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex space-x-2">
                        <button
                          onClick={() => openModal('infraction', infraction)}
                          className="text-blue-600 hover:text-blue-900"
                        >
                          <Edit className="w-4 h-4" />
                        </button>
                        <button
                          onClick={() => deleteItem('infraction', infraction.id)}
                          className="text-red-600 hover:text-red-900"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      );

      const ReportsTab = () => {
        const clubReport = generateClubReport();
        return (
          <div className="bg-white rounded-lg shadow-md p-6">
            <h2 className="text-xl font-bold mb-4">Relatórios</h2>
            <div className="mb-8">
              <h3 className="text-lg font-semibold mb-2">Classificação por Clube</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead className="bg-gray-50">
                    <tr>
                      <th className="px-4 py-2 text-left">Clube</th>
                      <th className="px-4 py-2 text-left">Atletas</th>
                      <th className="px-4 py-2 text-left">Pontos</th>
                      <th className="px-4 py-2 text-left">Ouros</th>
                    </tr>
                  </thead>
                  <tbody>
                    {clubReport.map((club, index) => (
                      <tr key={index} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                        <td className="px-4 py-2">{club.club}</td>
                        <td className="px-4 py-2">{club.athletes}</td>
                        <td className="px-4 py-2">{club.points}</td>
                        <td className="px-4 py-2">{club.gold}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
            <div>
              <h3 className="text-lg font-semibold mb-2">Ranking Individual</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead className="bg-gray-50">
                    <tr>
                      <th className="px-4 py-2 text-left">Posição</th>
                      <th className="px-4 py-2 text-left">Atleta</th>
                      <th className="px-4 py-2 text-left">Clube</th>
                      <th className="px-4 py-2 text-left">Pontos</th>
                    </tr>
                  </thead>
                  <tbody>
                    {getAthleteRanking().map((athlete, index) => (
                      <tr key={athlete.id} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                        <td className="px-4 py-2">{index + 1}º</td>
                        <td className="px-4 py-2">{athlete.name}</td>
                        <td className="px-4 py-2">{athlete.club}</td>
                        <td className="px-4 py-2">{athlete.points}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        );
      };

      return (
        <div className="min-h-screen bg-gray-100">
          <header className="bg-blue-800 text-white p-4 shadow-lg">
            <div className="container mx-auto">
              <h1 className="text-2xl font-bold flex items-center">
                <Trophy className="mr-2" />
                Sistema de Controle de Competição de Natação
              </h1>
              <p className="text-blue-200 mt-1">Gestão completa de competições aquáticas</p>
            </div>
          </header>
          <nav className="bg-white shadow-md">
            <div className="container mx-auto px-4">
              <div className="flex space-x-8">
                {[
                  { id: 'dashboard', label: 'Dashboard', icon: Trophy },
                  { id: 'athletes', label: 'Atletas', icon: Users },
                  { id: 'events', label: 'Provas', icon: Flag },
                  { id: 'registrations', label: 'Inscrições', icon: FileText },
                  { id: 'results', label: 'Resultados', icon: Medal },
                  { id: 'training', label: 'Treinos', icon: Clock },
                  { id: 'control', label: 'Banco', icon: CheckCircle },
                  { id: 'judiciary', label: 'Judiciária', icon: AlertCircle },
                  { id: 'reports', label: 'Relatórios', icon: BarChart2 }
                ].map(tab => (
                  <button
                    key={tab.id}
                    onClick={() => setActiveTab(tab.id)}
                    className={`flex items-center py-4 px-2 border-b-2 transition-colors ${
                      activeTab === tab.id
                        ? 'border-blue-600 text-blue-600'
                        : 'border-transparent text-gray-600 hover:text-blue-600'
                    }`}
                  >
                    <tab.icon className="w-4 h-4 mr-1" />
                    <span>{tab.label}</span>
                  </button>
                ))}
              </div>
            </div>
          </nav>
          <main className="container mx-auto px-4 py-8">
            {activeTab === 'dashboard' && <DashboardTab />}
            {activeTab === 'athletes' && <AthletesTab />}
            {activeTab === 'events' && <EventsTab />}
            {activeTab === 'registrations' && <RegistrationsTab />}
            {activeTab === 'results' && <ResultsTab />}
            {activeTab === 'training' && <TrainingTab />}
            {activeTab === 'control' && <ControlDesk />}
            {activeTab === 'judiciary' && <JudiciaryTab />}
            {activeTab === 'reports' && <ReportsTab />}
          </main>
          <Modal />
        </div>
      );
    };

    ReactDOM.render(<SwimmingCompetitionSystem />, document.getElementById('root'));
  </script>
</body>
</html>
