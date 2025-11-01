5. CÓDIGO FONTE COMPLETO DO PROTÓTIPO

Este documento contém o código-fonte dos principais componentes da aplicação React/TypeScript, focada na gestão de estoque e vendas da mercearia.

5.1 App.tsx (Componente Principal e Lógica de Estado)

Este componente é o container principal, responsável por gerenciar a autenticação (isLoggedIn), a navegação entre as páginas (currentPage) e o estado global dos dados (Produtos e Vendas), utilizando localStorage para simular a persistência de dados.

import { useState, useEffect } from "react";
import { LoginPage } from "./components/LoginPage";
import { DashboardPage } from "./components/DashboardPage";
// ... (demais imports de páginas)
import { Button } from "./components/ui/button";
import { LogOut } from "lucide-react";
import { Toaster } from "./components/ui/sonner";
import { toast } from "sonner@2.0.3";

// Interfaces de Tipagem
export interface Product {
 id: string;
 name: string;
 category: string;
 price: number;
 marketPrice: number;
 quantity: number;
 expiryDate: string;
}

export interface Sale {
 id: string;
 productId: string;
 productName: string;
 quantity: number;
 price: number;
 total: number;
 date: string;
}

type Page = "login" | "dashboard" | "register" | "inventory" | 
            "reports" | "sales" | "alerts" | "expiring";

export default function App() {
  const [currentPage, setCurrentPage] = useState("login");
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  
  // Estado e persistência de Produtos
  const [products, setProducts] = useState<Product[]>(() => {
    const saved = localStorage.getItem("products");
    return saved ? JSON.parse(saved) : [];
  });
  
  // Estado e persistência de Vendas
  const [sales, setSales] = useState<Sale[]>(() => {
    const saved = localStorage.getItem("sales");
    return saved ? JSON.parse(saved) : [];
  });

  // Efeitos colaterais para salvar no LocalStorage sempre que 'products' ou 'sales' mudam
  useEffect(() => {
    localStorage.setItem("products", JSON.stringify(products));
  }, [products]);

  useEffect(() => {
    localStorage.setItem("sales", JSON.stringify(sales));
  }, [sales]);

  const handleLogin = () => {
    setIsLoggedIn(true);
    setCurrentPage("dashboard");
  };

  const handleLogout = () => {
    setIsLoggedIn(false);
    setCurrentPage("login");
  };

  const handleNavigate = (page: Page) => {
    setCurrentPage(page);
  };
  
  const handleAddProduct = (product: Omit<Product, "id">) => {
    const newProduct = { ...product, id: Date.now().toString() };
    setProducts([...products, newProduct]);
  };

  const handleUpdateProduct = (id: string, updatedProduct: Omit<Product, "id">) => {
    setProducts(products.map((p) => 
      (p.id === id ? { ...updatedProduct, id } : p)));
  };

  const handleRemoveProduct = (id: string) => {
    setProducts(products.filter((p) => p.id !== id));
  };
  
  const handleAddSale = (sale: Omit<Sale, "id" | "date" | "productName"> & { productName: string }) => {
    const newSale: Sale = {
      ...sale,
      id: Date.now().toString(),
      date: new Date().toISOString(),
    };
    setSales([...sales, newSale]);
    setProducts(products.map((p) =>
      p.id === sale.productId ? 
        { ...p, quantity: p.quantity - sale.quantity } : p
    ));
  };
  
  // Renderiza a página de login se não estiver autenticado
  if (!isLoggedIn) {
    return <LoginPage onLogin={handleLogin} />;
  }

  // Layout principal da aplicação
  return (
    <div className="min-h-screen bg-gray-50 flex flex-col">
      <header className="flex items-center justify-between p-4 bg-white shadow-md">
        <div 
          className="text-xl font-bold text-green-700 cursor-pointer"
          onClick={() => handleNavigate("dashboard")}>
          Sistema de Estoque
        </div>

        <nav className="flex space-x-4">
          <Button 
            variant="ghost" 
            onClick={handleLogout}
            className="flex items-center space-x-2 text-red-600 hover:bg-red-50"
          >
            <LogOut className="w-4 h-4" />
            <span>Sair</span>
          </Button>
        </nav>
      </header>

      <main className="flex-1 p-6">
        {/* Renderização condicional das páginas - Aqui seria a lógica do switch */}
        {/* ... */}
      </main>

      <Toaster position="bottom-right" />
    </div>
  );
}


5.2 LoginPage.tsx (Componente de Login)

Este componente renderiza o formulário de login e chama a função de autenticação fornecida pelo App.tsx.

import { useState } from "react";
import { Input } from "./ui/input";
import { Button } from "./ui/button";
import { Label } from "./ui/label";

interface LoginPageProps {
 onLogin: () => void;
}

export function LoginPage({ onLogin }: LoginPageProps) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Simulação simples de login para o protótipo
    if (username && password) {
      onLogin();
    }
  };

  return (
    <div className="flex items-center justify-center min-h-screen bg-gray-100">
      <div className="w-full max-w-md p-8 space-y-6 bg-white rounded-lg shadow-xl">
        <div className="text-center">
          <h1 className="text-3xl font-bold text-green-800">
            Sistema de Estoque da Mercearia
          </h1>
          <p className="text-gray-500 mt-2">
            Acesso administrativo
          </p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="username">Usuário:</Label>
            <Input
              id="username"
              type="text"
              placeholder="Digite seu usuário"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">Senha:</Label>
            <Input
              id="password"
              type="password"
              placeholder="••••••••"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />
          </div>

          <Button type="submit" className="w-full bg-green-600 hover:bg-green-700">
            Entrar
          </Button>
        </form>
      </div>
    </div>
  );
}
