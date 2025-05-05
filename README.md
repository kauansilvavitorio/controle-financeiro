import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { PieChart, Pie, Cell, Tooltip, ResponsiveContainer } from "recharts";
import { format } from "date-fns";

const COLORS = ["#4ade80", "#f87171"]; // verde (entrada), vermelho (saída)

export default function FinanceDashboard() {
  const [transactions, setTransactions] = useState([]);
  const [form, setForm] = useState({ type: "entrada", value: "", name: "", date: "", scheduled: false });
  const [filters, setFilters] = useState({ name: "", date: "" });

  // Adicionar transação (com possível agendamento)
  const handleAddTransaction = () => {
    if (!form.value || !form.name || !form.date) return;
    const transaction = {
      ...form,
      value: parseFloat(form.value),
      scheduled: new Date(form.date) > new Date(),
    };
    setTransactions([...transactions, transaction]);
    setForm({ type: "entrada", value: "", name: "", date: "", scheduled: false });
  };

  // Atualiza transações agendadas que já passaram da data
  useEffect(() => {
    const updated = transactions.map(t => {
      if (t.scheduled && new Date(t.date) <= new Date()) {
        return { ...t, scheduled: false };
      }
      return t;
    });
    setTransactions(updated);
  }, [transactions]);

  const filteredTransactions = transactions.filter((t) => {
    const matchName = t.name.toLowerCase().includes(filters.name.toLowerCase());
    const matchDate = filters.date ? t.date === filters.date : true;
    return matchName && matchDate;
  });

  const total = transactions.reduce((acc, t) => t.scheduled ? acc : (t.type === "entrada" ? acc + t.value : acc - t.value), 0);

  const chartData = [
    { name: "Entradas", value: transactions.filter(t => t.type === "entrada" && !t.scheduled).reduce((a, b) => a + b.value, 0) },
    { name: "Saídas", value: transactions.filter(t => t.type === "saida" && !t.scheduled).reduce((a, b) => a + b.value, 0) },
  ];

  return (
    <div className="p-6 max-w-4xl mx-auto space-y-6">
      <h1 className="text-3xl font-bold text-center">💰 Controle Financeiro</h1>

      {/* Formulário */}
      <Card>
        <CardContent className="p-4 grid grid-cols-1 md:grid-cols-6 gap-4">
          <select
            value={form.type}
            onChange={(e) => setForm({ ...form, type: e.target.value })}
            className="border rounded p-2"
          >
            <option value="entrada">Entrada</option>
            <option value="saida">Saída</option>
          </select>
          <Input
            placeholder="Valor"
            type="number"
            value={form.value}
            onChange={(e) => setForm({ ...form, value: e.target.value })}
          />
          <Input
            placeholder="Nome"
            value={form.name}
            onChange={(e) => setForm({ ...form, name: e.target.value })}
          />
          <Input
            type="date"
            value={form.date}
            onChange={(e) => setForm({ ...form, date: e.target.value })}
          />
          <Button onClick={handleAddTransaction}>Adicionar</Button>
        </CardContent>
      </Card>

      {/* Filtros */}
      <Card>
        <CardContent className="p-4 flex flex-col md:flex-row gap-4">
          <Input
            placeholder="Filtrar por nome"
            value={filters.name}
            onChange={(e) => setFilters({ ...filters, name: e.target.value })}
          />
          <Input
            type="date"
            value={filters.date}
            onChange={(e) => setFilters({ ...filters, date: e.target.value })}
          />
        </CardContent>
      </Card>

      {/* Saldo total */}
      <Card>
        <CardContent className="p-4 text-xl font-semibold text-center">
          Saldo Total: <span className={total >= 0 ? "text-green-500" : "text-red-500"}>R$ {total.toFixed(2)}</span>
        </CardContent>
      </Card>

      {/* Gráfico */}
      <Card>
        <CardContent className="h-64">
          <ResponsiveContainer width="100%" height="100%">
            <PieChart>
              <Pie
                data={chartData}
                dataKey="value"
                nameKey="name"
                cx="50%"
                cy="50%"
                outerRadius={80}
                fill="#8884d8"
                label
              >
                {chartData.map((_, index) => (
                  <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                ))}
              </Pie>
              <Tooltip />
            </PieChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Lista de transações */}
      <Card>
        <CardContent className="p-4">
          <h2 className="text-xl font-bold mb-2">Transações</h2>
          <ul className="space-y-2">
            {filteredTransactions.map((t, i) => (
              <li
                key={i}
                className="flex justify-between border-b pb-1 text-sm md:text-base"
              >
                <span>{t.name}</span>
                <span>{format(new Date(t.date), "dd/MM/yyyy")}</span>
                <span className={t.type === "entrada" ? "text-green-500" : "text-red-500"}>R$ {t.value.toFixed(2)}</span>
                {t.scheduled && <span className="text-yellow-500 ml-2">(Agendada)</span>}
              </li>
            ))}
            {filteredTransactions.length === 0 && <p className="text-gray-500">Nenhuma transação encontrada.</p>}
          </ul>
        </CardContent>
      </Card>
    </div>
  );
}
