import React, { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import {
  Menu,
  Search,
  Sun,
  Moon,
  LogIn,
  LogOut,
  PlusCircle,
  Edit,
  Trash2,
  CheckCircle,
  Loader2,
  AlertTriangle,
  BookOpen,
  X,
  Flag,
  Info
} from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import {
  Sheet,
  SheetContent,
  SheetHeader,
  SheetTitle,
  SheetDescription,
  SheetFooter,
  SheetTrigger,
} from '@/components/ui/sheet';
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from "@/components/ui/select"
import { cn } from '@/lib/utils';

// Mock data för initial setup (ersätt med Firebase data senare)
const initialNews = [
  {
    id: '1',
    title: 'Visby firar 800 år med stor festival',
    content:
      'Visby firar i år sitt 800-årsjubileum med en stor festival som pågår hela sommaren. Det blir konserter, teaterföreställningar, historiska återskapanden och mycket mer. Missa inte!',
    category: 'Evenemang',
    published: true,
    createdAt: new Date(2024, 7, 10),
  },
  {
    id: '2',
    title: 'Nytt bostadsområde planeras i Hemse',
    content:
      'Ett nytt bostadsområde med både hyresrätter och bostadsrätter planeras i Hemse. Bygget väntas starta nästa år och kommer att bidra till att lösa bostadsbristen på södra Gotland.',
    category: 'Samhälle',
    published: true,
    createdAt: new Date(2024, 7, 8),
  },
  {
    id: '3',
    title: 'Debatt om vindkraftverk på norra Gotland',
    content:
      'Frågan om att bygga vindkraftverk på norra Gotland fortsätter att skapa debatt. Förespråkare lyfter fram förnybar energi, medan motståndare oroar sig för påverkan på landskapet och djurlivet.',
    category: 'Miljö',
    published: false,
    createdAt: new Date(2024, 7, 5),
  },
    {
    id: '4',
    title: 'Politisk debatt om regionens budget',
    content: 'Region Gotlands politiker debatterar den kommande budgeten. Oenighet råder om hur resurserna ska fördelas mellan olika verksamheter som skola, vård och infrastruktur.',
    category: 'Politik',
    published: true,
    createdAt: new Date(2024, 7, 12),
  },
  {
    id: '5',
    title: 'Stor ökning av turister från Tyskland',
    content: 'Antalet turister från Tyskland har ökat markant denna sommar. Många lockas av Gotlands vackra natur, historiska sevärdheter och unika kultur.',
    category: 'Turism',
    published: true,
    createdAt: new Date(2024, 7, 11),
  },
  {
        id: '6',
        title: 'Kulturfestival i Visby lockar stor publik',
        content: 'Den årliga kulturfestivalen i Visby lockade i år en rekordstor publik. Festivalen bjöd på en mängd olika evenemang, inklusive konserter, teaterföreställningar och konstutställningar.',
        category: 'Evenemang',
        published: true,
        createdAt: new Date(2024, 7, 15),
    },
    {
        id: '7',
        title: 'Nya restriktioner för vattenanvändning införs',
        content: 'På grund av den långvariga torkan har Region Gotland beslutat att införa nya restriktioner för vattenanvändning. Invånarna uppmanas att spara vatten och vara försiktiga med sin förbrukning.',
        category: 'Samhälle',
        published: true,
        createdAt: new Date(2024, 7, 14),
    },
    {
        id: '8',
        title: 'Protest mot planerad gruva på södra Gotland',
        content: 'En stor protestaktion hölls i helgen mot planerna på att öppna en ny gruva på södra Gotland. Demonstranterna uttryckte oro för miljökonsekvenserna och krävde att planerna stoppas.',
        category: 'Miljö',
        published: false,
        createdAt: new Date(2024, 7, 13),
    },
];

const categories = ['Politik', 'Samhälle', 'Evenemang', 'Miljö', 'Turism'];

// Animation Variants
const newsCardVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.3 } },
  exit: { opacity: 0, y: -20, transition: { duration: 0.2 } },
};

const articleVariants = {
    hidden: { opacity: 0, x: 50 },
    visible: { opacity: 1, x: 0, transition: { duration: 0.5, ease: "easeInOut" } },
    exit: { opacity: 0, x: -50, transition: { duration: 0.3 } }
};

// Helper function for Text-to-Speech
const speakText = (text: string) => {
  if ('speechSynthesis' in window) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'sv-SE'; // Set language to Swedish
    speechSynthesis.speak(utterance);
  } else {
    console.warn('Text-to-Speech is not supported in this browser.');
  }
};

const O_Nytt = () => {
  const [news, setNews] = useState(initialNews);
  const [selectedArticle, setSelectedArticle] = useState<typeof initialNews[0] | null>(null);
  const [isDarkMode, setIsDarkMode] = useState(true); // Default to dark mode
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [user, setUser] = useState<{ uid: string; email: string } | null>(null); // Mock user
  const [isAdmin, setIsAdmin] = useState(false); // Default to false, set to true on correct login
  const [isCreating, setIsCreating] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [newArticle, setNewArticle] = useState<Partial<typeof initialNews[0]>>({
    title: '',
    content: '',
    category: 'Samhälle', // Default category
    published: false,
  });
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [showAbout, setShowAbout] = useState(false);
  const [loginPassword, setLoginPassword] = useState('');


    // Mock authentication and data functions (ersätt med Firebase)
    const mockAuth = {
        signIn: async (email: string, password: string) => { // Added password
            setLoading(true);
            setError(null);
              // Simulate login delay
              await new Promise((resolve) => setTimeout(resolve, 1000));

              if (password === 'snoxizz653') { // Changed condition
                  const mockUser = { uid: 'mock-user-id', email: email };
                  setUser(mockUser);
                  setIsAdmin(true); // Set isAdmin to true on successful login
                  return mockUser; // Return the mock user object
              } else {
                  setError('Felaktigt lösenord.'); // Changed error message
                  return null;
              }
        },
        signOut: async () => {
            setLoading(true);
            await new Promise(resolve => setTimeout(resolve, 500));
            setUser(null);
            setIsAdmin(false);
            setLoading(false);
        },
        currentUser: () => {
            // In a real app, this would check for an existing session
            return null; // For this example, we start with no user logged in
        },
    };

    const mockFirestore = {
        collection: () => ({
            orderBy: () => ({
                get: async () => {
                    // Simulate fetching data
                    await new Promise((resolve) => setTimeout(resolve, 500));
                    // Convert mock data to the format expected by the app
                    const snapshot = initialNews.map(doc => ({
                        id: doc.id,
                        data: () => doc
                    }));
                    return { docs: snapshot };
                },
            }),
            doc: (id: string) => ({
                set: async (data: any) => {
                    // Simulate setting data
                    await new Promise(resolve => setTimeout(resolve, 200));
                    if (news.find(n => n.id === id)) {
                        setNews(prevNews => prevNews.map(n => n.id === id ? { ...n, ...data } : n));
                    } else {
                        setNews(prevNews => [...prevNews, { id, ...data }]);
                    }
                },
                delete: async () => {
                    await new Promise(resolve => setTimeout(resolve, 200));
                    setNews(prevNews => prevNews.filter(n => n.id !== id));
                }
            }),
            add: async (data: any) => {
                await new Promise(resolve => setTimeout(resolve, 200));
                const newId = String(Math.random().toString(36).substring(2, 9)); // Generate a simple ID.  Use a proper ID generator in real app.
                const newArticleWithId = { ...data, id: newId, createdAt: new Date() };
                setNews(prevNews => [...prevNews, newArticleWithId]);
                return { id: newId }; // Mock the return of a document reference
            }
        }),
    };

  // Fetch initial news data (ersätt med faktisk datahämtning)
    useEffect(() => {
        const fetchNews = async () => {
            setLoading(true);
            try {
                // Mock fetching
                const snapshot = await mockFirestore.collection('news').orderBy('createdAt').get();
                const fetchedNews = snapshot.docs.map(doc => ({
                    id: doc.id,
                    ...doc.data(),
                }));
                setNews(fetchedNews);
            } catch (err: any) {
                setError(err.message || 'Failed to fetch news.');
            } finally {
                setLoading(false);
            }
        };

        fetchNews();
    }, []);

    // Mock authentication state persistence
    useEffect(() => {
        const storedUser = localStorage.getItem('oNyttUser');
        if (storedUser) {
            setUser(JSON.parse(storedUser));
            setIsAdmin(true); //  set admin status.
        }
    }, []);

    useEffect(() => {
        if (user) {
            localStorage.setItem('oNyttUser', JSON.stringify(user));
        } else {
            localStorage.removeItem('oNyttUser');
        }
    }, [user]);

      const handleLogin = async (email: string, password: string) => { // Added password parameter
          try {
              const loggedInUser = await mockAuth.signIn(email, password); // Pass password
              if (loggedInUser) {
              setUser(loggedInUser);
              setIsAdmin(true); //  set admin status.
              }
          } catch (err: any) {
              setError(err.message);
          } finally {
              setLoading(false);
          }
      };

      const handleLogout = async () => {
          try {
              await mockAuth.signOut();
              setUser(null);
              setIsAdmin(false);
          } catch (err: any) {
              setError(err.message);
          }
      };

      const handleCreateArticle = async () => {
          if (!newArticle.title || !newArticle.content || !newArticle.category) {
              setError("Vänligen fyll i alla fält.");
              return;
          }

          setLoading(true);
          try {
              const docRef = await mockFirestore.collection('news').add({
                  ...newArticle,
                  published: false, // Start as draft
                  createdAt: new Date(),
              });
              const createdArticle = { ...newArticle, id: docRef.id, published: false, createdAt: new Date() }; // Add the ID and published status
                setNews(prevNews => [...prevNews, createdArticle as typeof initialNews[0]]);  //  add to local state
              setNewArticle({ title: '', content: '', category: 'Samhälle' }); // Reset form
              setIsCreating(false); // Close the form
              setError(null);
          } catch (err: any) {
              setError(err.message || 'Failed to create article.');
          } finally {
              setLoading(false);
          }
      };

      const handlePublishArticle = async (id: string) => {
          setLoading(true);
          try {
              const articleToUpdate = news.find((article) => article.id === id);
                if (articleToUpdate) {
                    await mockFirestore.collection('news').doc(id).set({
                        ...articleToUpdate,
                        published: true,
                    });
                    setNews((prevNews) =>
                        prevNews.map((article) =>
                        article.id === id ? { ...article, published: true } : article
                        )
                    );
                }

          } catch (err: any) {
              setError(err.message || 'Failed to publish article.');
          } finally {
              setLoading(false);
          }
      };

  const handleEditArticle = async (id: string) => {
    const articleToEdit = news.find((article) => article.id === id);
    if (articleToEdit) {
      setNewArticle(articleToEdit);
      setIsEditing(true);
      setSelectedArticle(null); // Close article view
    }
  };

    const handleUpdateArticle = async (id: string) => {
        setLoading(true);
        try {
            await mockFirestore.collection('news').doc(id).set({
                ...newArticle,
                id,
            });
              setNews(prevNews =>
                  prevNews.map(article =>
                      article.id === id ? { ...article, ...newArticle } : article
                  )
              );
            setIsEditing(false);
            setNewArticle({ title: '', content: '', category: 'Samhälle' });
            setError(null);
        } catch (err: any) {
            setError(err.message || 'Failed to update article.');
        } finally {
            setLoading(false);
        }
    };

  const handleDeleteArticle = async (id: string) => {
    setLoading(true);
    try {
      await mockFirestore.collection('news').doc(id).delete();
      setNews((prevNews) => prevNews.filter((article) => article.id !== id));
      if (selectedArticle?.id === id) {
        setSelectedArticle(null); // Clear selected article if it's deleted
      }
    } catch (err: any) {
      setError(err.message || 'Failed to delete article.');
    } finally {
      setLoading(false);
    }
  };

    const filteredNews = news.filter((article) =>
        article.published && (article.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
        article.content.toLowerCase().includes(searchTerm.toLowerCase()))
    );

  return (
    <div className={cn(
        'min-h-screen bg-background text-foreground',
        isDarkMode ? 'dark' : ''
    )}>
      {/* Fixed Header */}
      <header className="sticky top-0 z-50 bg-background border-b border-border/40 backdrop-blur-lg">
        <div className="container flex items-center justify-between h-16">
          {/* Logo */}
          <div className="flex items-center">
            <span className="font-bold text-xl">
              <span className="text-blue-500">Ö</span>-nytt
            </span>
              <Flag className="w-5 h-5 ml-1 text-yellow-500" />
          </div>

          {/* Search Bar */}
            <div className="flex-1 mx-4 max-w-md">
                <Input
                type="text"
                placeholder="Sök nyheter..."
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                className="w-full"
                />
            </div>

          {/* Menu (for small screens) */}
          <Sheet open={isMenuOpen} onOpenChange={setIsMenuOpen}>
            <SheetTrigger asChild>
              <Button variant="ghost" size="icon" className="md:hidden">
                <Menu className="h-6 w-6" />
              </Button>
            </SheetTrigger>
            <SheetContent side="left" className="w-full sm:w-[300px]">
              <SheetHeader>
                <SheetTitle>Meny</SheetTitle>
                <SheetDescription>
                  Navigera till olika kategorier och funktioner.
                </SheetDescription>
              </SheetHeader>
              <div className="mt-4 space-y-4">
                {/* Navigation Links */}
                <nav>
                  <ul className="space-y-2">
                    {categories.map((category) => (
                      <li key={category}>
                        <Button
                          variant="ghost"
                          className="w-full justify-start"
                          onClick={() => {
                            // Handle category selection (e.g., filter news)
                            setIsMenuOpen(false); // Close menu after selection
                          }}
                        >
                          {category}
                        </Button>
                      </li>
                    ))}
                    <li>
                        <Button
                            variant="ghost"
                            className="w-full justify-start"
                            onClick={() => {
                                setShowAbout(true);
                                setIsMenuOpen(false)
                            }}
                        >
                            <Info className="mr-2 h-4 w-4" /> Om Ö-nytt
                        </Button>
                    </li>
                  </ul>
                </nav>
                {/* Dark/Light Mode Toggle */}
                <Button
                  variant="ghost"
                  className="w-full justify-start"
                  onClick={() => setIsDarkMode((prev) => !prev)}
                >
                  {isDarkMode ? (
                    <>
                      <Sun className="mr-2 h-4 w-4" /> Ljust Läge
                    </>
                  ) : (
                    <>
                      <Moon className="mr-2 h-4 w-4" /> Mörkt Läge
                    </>
                  )}
                </Button>
                {/* Login/Logout */}
                {user ? (
                  <Button
                    variant="ghost"
                    className="w-full justify-start"
                    onClick={handleLogout}
                  >
                    <LogOut className="mr-2 h-4 w-4" /> Logga ut
                  </Button>
                ) : (
                  <Sheet>
                    <SheetTrigger asChild>
                      <Button variant="ghost" className="w-full justify-start">
                        <LogIn className="mr-2 h-4 w-4" /> Logga in
                      </Button>
                    </SheetTrigger>
                    <SheetContent>
                      <SheetHeader>
                        <SheetTitle>Logga in</SheetTitle>
                        <SheetDescription>
                          Logga in med ditt lösenord för att hantera nyheter.
                        </SheetDescription>
                      </SheetHeader>
                      <div className="mt-4 space-y-4">
                        <Input
                            type="password"
                            placeholder="Lösenord"
                            value={loginPassword}
                            onChange={(e) => setLoginPassword(e.target.value)}
                            onKeyDown={(e) => {
                                if (e.key === 'Enter') {
                                    handleLogin("anton@example.com", loginPassword); // Hardcoded email
                                }
                            }}
                        />
                        <Button
                            className="w-full"
                            onClick={() => {
                                handleLogin("anton@example.com", loginPassword); // Hardcoded email
                            }}
                            disabled={loading}
                        >
                            {loading ? (
                            <>
                                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                                Loggar in...
                            </>
                            ) : (
                            'Logga in'
                            )}
                        </Button>
                         {error && (
                                                <p className="text-red-500 text-sm">
                                                    {error}
                                                </p>
                                            )}
                      </div>
                    </SheetContent>
                  </Sheet>
                )}
              </div>
            </SheetContent>
          </Sheet>

          {/* Desktop Navigation */}
          <nav className="hidden md:block">
            <ul className="flex items-center gap-6">
              {categories.map((category) => (
                <li key={category}>
                  <Button
                    variant="ghost"
                    onClick={() => {
                      // Handle category selection
                    }}
                  >
                    {category}
                  </Button>
                </li>
              ))}
              <li>
                <Button
                    variant="ghost"
                    onClick={() => setShowAbout(true)}
                >
                    <Info className="h-4 w-4" /> Om Ö-nytt
                </Button>
              </li>
              {/* Dark/Light Mode Toggle */}
              <li>
                <Button
                  variant="ghost"
                  onClick={() => setIsDarkMode((prev) => !prev)}
                >
                  {isDarkMode ? (
                    <>
                      <Sun className="h-4 w-4" />
                    </>
                  ) : (
                    <>
                      <Moon className="h-4 w-4" />
                    </>
                  )}
                </Button>
              </li>
              {/* Login/Logout */}
              <li>
                {user ? (
                  <Button variant="ghost" onClick={handleLogout}>
                    <LogOut className="h-4 w-4" />
                  </Button>
                ) : (
                  <Sheet>
                    <SheetTrigger asChild>
                      <Button variant="ghost">
                        <LogIn className="h-4 w-4" />
                      </Button>
                    </SheetTrigger>
                    <SheetContent>
                      <SheetHeader>
                        <SheetTitle>Logga in</SheetTitle>
                        <SheetDescription>
                          Logga in med ditt lösenord för att hantera nyheter.
                        </SheetDescription>
                      </SheetHeader>
                      <div className="mt-4 space-y-4">
                        <Input
                            type="password"
                            placeholder="Lösenord"
                            value={loginPassword}
                            onChange={(e) => setLoginPassword(e.target.value)}
                            onKeyDown={(e) => {
                                if (e.key === 'Enter') {
                                    handleLogin("anton@example.com", loginPassword);
                                }
                            }}
                        />
                        <Button
                            className="w-full"
                            onClick={() => {
                                handleLogin("anton@example.com", loginPassword);
                            }}
                            disabled={loading}
                        >
                            {loading ? (
                            <>
                                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                                Loggar in...
                            </>
                            ) : (
                            'Logga in'
                            )}
                        </Button>
                         {error && (
                                                <p className="text-red-500 text-sm">
                                                    {error}
                                                </p>
                                            )}
                      </div>
                    </SheetContent>
                  </Sheet>
                )}
              </li>
            </ul>
          </nav>
        </div>
      </header>

      {/* Main Content */}
      <main className="container py-8">
        {/* Admin Panel */}
        {isAdmin && (
          <div className="mb-8">
            <h2 className="text-2xl font-bold mb-4">Adminpanel</h2>
            <div className="flex flex-wrap gap-4">
              <Button onClick={() => setIsCreating(true)} disabled={loading}>
                <PlusCircle className="mr-2h-4 w-4" /> Skapa Ny Artikel
              </Button>
              {isCreating && (
                <div className="bg-card p-6 rounded-lg shadow-md">
                  <h3 className="text-lg font-semibold mb-4">Skapa Ny Artikel</h3>
                  <div className="space-y-4">
                    <Input
                      placeholder="Titel"
                      value={newArticle.title || ''}
                      onChange={(e) => setNewArticle({ ...newArticle, title: e.target.value })}
                    />
                    <Textarea
                      placeholder="Innehåll"
                      value={newArticle.content || ''}
                      onChange={(e) => setNewArticle({ ...newArticle, content: e.target.value })}
                    />
                    <Select
                        onValueChange={(value) => setNewArticle({ ...newArticle, category: value })}
                        defaultValue={newArticle.category}
                    >
                        <SelectTrigger className="w-full">
                            <SelectValue placeholder="Kategori" />
                        </SelectTrigger>
                        <SelectContent>
                            {categories.map(cat => (
                                <SelectItem key={cat} value={cat}>{cat}</SelectItem>
                            ))}
                        </SelectContent>
                    </Select>
                    <div className="flex gap-4">
                        <Button onClick={handleCreateArticle} disabled={loading}>
                        {loading ? (
                            <>
                            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                            Skapar...
                            </>
                        ) : (
                            'Skapa'
                        )}
                        </Button>
                        <Button variant="outline" onClick={() => {
                            setIsCreating(false);
                            setIsEditing(false);
                            setNewArticle({title: '', content: '', category: 'Samhälle'})
                            }}>
                            Avbryt
                        </Button>
                    </div>
                     {error && (
                                                <p className="text-red-500 text-sm">
                                                    {error}
                                                </p>
                                            )}
                  </div>
                </div>
              )}
              {isEditing && selectedArticle && (
                <div className="bg-card p-6 rounded-lg shadow-md">
                  <h3 className="text-lg font-semibold mb-4">Redigera Artikel</h3>
                  <div className="space-y-4">
                    <Input
                      placeholder="Titel"
                      value={newArticle.title || ''}
                      onChange={(e) => setNewArticle({ ...newArticle, title: e.target.value })}
                    />
                    <Textarea
                      placeholder="Innehåll"
                      value={newArticle.content || ''}
                      onChange={(e) => setNewArticle({ ...newArticle, content: e.target.value })}
                    />
                    <Select
                        onValueChange={(value) => setNewArticle({ ...newArticle, category: value })}
                        defaultValue={newArticle.category}
                    >
                        <SelectTrigger className="w-full">
                            <SelectValue placeholder="Kategori" />
                        </SelectTrigger>
                        <SelectContent>
                            {categories.map(cat => (
                                <SelectItem key={cat} value={cat}>{cat}</SelectItem>
                            ))}
                        </SelectContent>
                    </Select>
                    <div className="flex gap-4">
                      <Button onClick={() => handleUpdateArticle(selectedArticle.id)} disabled={loading}>
                        {loading ? (
                            <>
                            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                            Uppdaterar...
                            </>
                            ) : (
                            <><CheckCircle className="mr-2 h-4 w-4" /> Spara</>
                            )}
                      </Button>
                      <Button variant="outline" onClick={() => {
                        setIsEditing(false);
                        setNewArticle({ title: '', content: '', category: 'Samhälle' });
                        }}>
                        Avbryt
                      </Button>
                    </div>
                     {error && (
                                                <p className="text-red-500 text-sm">
                                                    {error}
                                                </p>
                                            )}
                  </div>
                </div>
              )}
            </div>
             <div className="mt-8">
                <h3 className="text-lg font-semibold mb-4">Hantera Artiklar</h3>
                 <div className="space-y-4">
                    {news.map(article => (
                         <div key={article.id} className="bg-card p-4 rounded-lg shadow-md flex items-center justify-between">
                            <div>
                                <h4 className="font-medium">{article.title}</h4>
                                <p className="text-sm text-muted-foreground">Kategori: {article.category}</p>
                                <p className="text-sm text-muted-foreground">Status: {article.published ? 'Publicerad' : 'Utkast'}</p>
                            </div>
                            <div className="flex gap-2">
                                <Button
                                    variant="outline"
                                    size="icon"
                                    onClick={() => handleEditArticle(article.id)}
                                    disabled={loading}
                                >
                                    <Edit className="h-4 w-4" />
                                </Button>
                                 <Button
                                    variant="destructive"
                                    size="icon"
                                    onClick={() => handleDeleteArticle(article.id)}
                                    disabled={loading}
                                >
                                    <Trash2 className="h-4 w-4" />
                                </Button>
                                {!article.published && (
                                    <Button
                                        variant="default"
                                        size="sm"
                                        onClick={() => handlePublishArticle(article.id)}
                                        disabled={loading}
                                    >
                                         {loading ? (
                                            <>
                                            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                                            Publicerar...
                                            </>
                                        ) : (
                                            'Publicera'
                                        )}
                                    </Button>
                                )}
                            </div>
                         </div>
                    ))}
                 </div>
             </div>
          </div>
        )}

        {/* News Feed */}
        <h2 className="text-3xl font-bold mb-6">Senaste Nyheter</h2>
        {loading && <p className="text-gray-500">Laddar nyheter...</p>}
        {error && <p className="text-red-500">{error}</p>}
        {!loading && !error && (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <AnimatePresence>
              {filteredNews.map((article) => (
                <motion.div
                  key={article.id}
                  variants={newsCardVariants}
                  initial="hidden"
                  animate="visible"
                  exit="exit"
                  className="bg-card rounded-lg shadow-md overflow-hidden transition-transform duration-300 hover:scale-[1.02] hover:shadow-lg"
                  onClick={() => setSelectedArticle(article)}
                >
                  <div className="p-6">
                    <h3 className="text-xl font-semibold mb-2">{article.title}</h3>
                    <p className="text-sm text-gray-500 mb-4">
                      {article.createdAt.toLocaleDateString('sv-SE')}
                    </p>
                    <p className="text-gray-700 line-clamp-3">{article.content}</p>
                  </div>
                </motion.div>
              ))}
            </AnimatePresence>
          </div>
        )}
         {/* Selected Article View */}
            <AnimatePresence>
                {selectedArticle && (
                    <motion.div
                        variants={articleVariants}
                        initial="hidden"
                        animate="visible"
                        exit="exit"
                        className="fixed inset-0 bg-black/50 z-40 flex items-center justify-center p-4 overflow-y-auto"
                    >
                        <div className="bg-card rounded-xl shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto relative">
                            <Button
                                variant="ghost"
                                size="icon"
                                className="absolute top-4 right-4 z-50"
                                onClick={() => setSelectedArticle(null)}
                            >
                                <X className="h-6 w-6" />
                            </Button>
                            <div className="p-8 space-y-4">
                                <h2 className="text-3xl font-bold">{selectedArticle.title}</h2>
                                <p className="text-sm text-gray-500">
                                    {selectedArticle.createdAt.toLocaleDateString('sv-SE')}
                                </p>
                                <p className="text-gray-700 whitespace-pre-line">{selectedArticle.content}</p>
                                 {/* Text-to-Speech Button */}
                                <Button
                                    variant="outline"
                                    onClick={() => speakText(selectedArticle.content)}
                                    className="mt-4"
                                >
                                    Lyssna
                                </Button>
                            </div>
                        </div>
                    </motion.div>
                )}
            </AnimatePresence>
             {/* About Ö-nytt Modal */}
            <Sheet open={showAbout} onOpenChange={setShowAbout}>
                <SheetContent>
                <SheetHeader>
                    <SheetTitle>Om Ö-nytt</SheetTitle>
                    <SheetDescription>
                    En nyhetsapp för Gotland, skapad av Anton Eliasson.
                    </SheetDescription>
                </SheetHeader>
                <div className="mt-4 space-y-4">
                    <p>
                    Välkommen till Ö-nytt, din lokala nyhetskälla för Gotland. Denna app är ett hobbyprojekt skapat av Anton Eliasson för att utforska React och Firebase.
                    </p>
                    <p>
                    Appen är tänkt att samla nyheter från hela Gotland, med fokus på lokala händelser och berättelser.
                    </p>
                     <p>
                     Funktionalitet:
                     </p>
                    <ul className="list-disc list-inside">
                        <li>Läs de senaste nyheterna från Gotland</li>
                        <li>Sök efter specifika nyheter</li>
                        <li>Välj mellan ljust och mörkt läge</li>
                        {user && (
                            <>
                            <li>Skapa nya artiklar (endast för administratörer)</li>
                            <li>Redigera och ta bort artiklar (endast för administratörer)</li>
                            <li>Publicera utkast (endast för administratörer)</li>
                            </>
                        )}

                    </ul>
                    <p>
                    Observera att detta är ett hobbyprojekt.
                    </p>
                     <p>
                        <strong>Teknik</strong>
                    </p>
                    <ul className="list-disc list-inside">
                        <li>React</li>
                        <li>Firebase (Firestore, Authentication)</li>
                        <li>Tailwind CSS</li>
                        <li>Framer Motion</li>
                        <li>shadcn/ui</li>
                    </ul>
                </div>
                <SheetFooter>
                    <Button onClick={() => setShowAbout(false)}>Stäng</Button>
                </SheetFooter>
                </SheetContent>
            </Sheet>
      </main>
    </div>
  );
};

export default O_Nytt;

