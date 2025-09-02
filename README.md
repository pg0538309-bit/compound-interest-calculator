import React, { useMemo, useState } from "react";
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  Tooltip,
  ResponsiveContainer,
  CartesianGrid,
  Legend,
} from "recharts";

// currency formatter
const fmtUSD = (n: number) =>
  n.toLocaleString(undefined, { style: "currency", currency: "USD", maximumFractionDigits: 0 });

export default function CompoundCalc() {
  // === Inputs ===
  const [grossSavings, setGrossSavings] = useState<number>(250000);
  const [taxRate, setTaxRate] = useState<number>(24); // %
  const [spending, setSpending] = useState<number>(50000); // absolute $ spending amount

  const [annualReturn, setAnnualReturn] = useState<number>(5); // % -- fixed at 5 by default
  const [years, setYears] = useState<number>(30);
  const [monthlyContribution, setMonthlyContribution] = useState<number>(0); // optional new contributions

  // === Derived amounts ===
  const afterTax = useMemo(() => Math.max(0, grossSavings * (1 - taxRate / 100)), [grossSavings, taxRate]);
  const investable = useMemo(() => Math.max(0, afterTax - spending), [afterTax, spending]);

  // === Projection logic: monthly compounding at annualReturn
  const data = useMemo(() => {
    const mRate = annualReturn / 100 / 12;
    const months = Math.max(1, Math.round(years * 12));
    let bal = investable; // starting principal is investable amount
    let contrib = 0;
    const out: Array<{ year: number; balance: number; contributions: number }> = [];

    for (let m = 1; m <= months; m++) {
      bal *= 1 + mRate; // growth
      bal += monthlyContribution; // add ongoing savings
      contrib += monthlyContribution;
      if (m % 12 === 0) {
        out.push({ year: m / 12, balance: bal, contributions: contrib });
      }
    }
    return out;
  }, [annualReturn, years, investable, monthlyContribution]);

  const final = data[data.length - 1];

  // Export projection data to CSV
  const exportCSV = () => {
    const headers = [
      "Year",
      "Balance",
      "TotalContributions",
      "StartInvestable",
      "AnnualReturn%",
      "Years",
      "MonthlyContribution",
      "TaxRate%",
      "GrossSavings",
      "Spending"
    ];
    const rows = data.map((d) => [
      d.year,
      Math.round(d.balance * 100) / 100,
      Math.round(d.contributions * 100) / 100,
      Math.round(investable * 100) / 100,
      annualReturn,
      years,
      monthlyContribution,
      taxRate,
      grossSavings,
      spending,
    ]);
    const csv = [headers.join(","), ...rows.map((r) => r.join(","))].join("\n");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    const ts = new Date().toISOString().slice(0,19).replace(/[:T]/g, "-");
    a.download = `compound_projection_${ts}.csv`;
    a.click();
    URL.revokeObjectURL(url);
  };

  // Export projection data to CSV
  const exportCSV = () => {
    const headers = [
      "Year",
      "Balance",
      "TotalContributions",
      "StartInvestable",
      "AnnualReturn%",
      "Years",
      "MonthlyContribution",
      "TaxRate%",
      "GrossSavings",
      "Spending"
    ];
    const rows = data.map((d) => [
      d.year,
      Math.round(d.balance * 100) / 100,
      Math.round(d.contributions * 100) / 100,
      Math.round(investable * 100) / 100,
      annualReturn,
      years,
      monthlyContribution,
      taxRate,
      grossSavings,
      spending,
    ]);
    const csv = [headers.join(","), ...rows.map((r) => r.join(","))].join("\n");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    const ts = new Date().toISOString().slice(0,19).replace(/[:T]/g, "-");
    a.download = `compound_projection_${ts}.csv`;
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="min-h-screen w-full bg-neutral-50 text-neutral-900 p-6">
      <div className="max-w-6xl mx-auto space-y-6">
        <header className="space-y-1">
          <h1 className="text-2xl md:text-3xl font-bold">Interactive Compound Interest Calculator</h1>
          <p className="text-neutral-600">Start with gross savings → apply your tax rate → subtract spending to get your investable amount. We then grow that amount at <strong>{annualReturn}%</strong> with monthly compounding. Adjust sliders to explore scenarios.</p>
        </header>

        {/* Inputs */}
        <section className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          <Card>
            <Label>Gross Savings</Label>
            <NumberInput value={grossSavings} onChange={setGrossSavings} step={1000} min={0} />
            <Range value={grossSavings} onChange={setGrossSavings} min={0} max={10_000_000} step={1000} />
            <SmallLine>Drag 0–$10,000,000</SmallLine>
          </Card>

          <Card>
            <Label>Tax Rate</Label>
            <NumberInput value={taxRate} onChange={setTaxRate} step={0.5} min={0} max={60} suffix="%" />
            <Range value={taxRate} onChange={setTaxRate} min={0} max={60} step={0.5} />
            <SmallLine>Effective rate applied to gross savings</SmallLine>
          </Card>

          <Card>
            <Label>Spending (one-time)</Label>
            <NumberInput value={spending} onChange={setSpending} step={100} min={0} />
            <Range value={spending} onChange={setSpending} min={0} max={10_000_000} step={1000} />
            <SmallLine>Amount you need to set aside now</SmallLine>
          </Card>

          <Card>
            <Label>Additional Monthly Contribution (optional)</Label>
            <NumberInput value={monthlyContribution} onChange={setMonthlyContribution} step={50} min={0} />
            <Range value={monthlyContribution} onChange={setMonthlyContribution} min={0} max={100_000} step={100} />
            <SmallLine>Add ongoing savings to boost growth</SmallLine>
          </Card>

          <Card>
            <Label>Time Horizon (years)</Label>
            <NumberInput value={years} onChange={setYears} step={1} min={1} max={60} />
            <Range value={years} onChange={setYears} min={1} max={60} step={1} />
            <SmallLine>Extend your investing timeline</SmallLine>
          </Card>

          <Card>
            <Label>Annual Return</Label>
            <NumberInput value={annualReturn} onChange={setAnnualReturn} step={0.25} min={0} max={15} suffix="%" />
            <Range value={annualReturn} onChange={setAnnualReturn} min={0} max={15} step={0.25} />
            <SmallLine>Default 5% (adjust to compare)</SmallLine>
          </Card>
        </section>

        {/* Snapshot */}
        <section className="grid grid-cols-1 md:grid-cols-4 gap-4">
          <StatCard title="After-Tax Amount" value={fmtUSD(afterTax)} sub="Gross minus taxes" />
          <StatCard title="Investable Now" value={fmtUSD(investable)} sub="After spending" />
          <StatCard title="Monthly Contribution" value={fmtUSD(monthlyContribution)} sub="Optional ongoing savings" />
          {final && (
            <StatCard title={`Balance in ${years} yrs`} value={fmtUSD(final.balance)} sub={`Total added over time ${fmtUSD(final.contributions)}`} />
          )}
        </section>

        {/* Chart */}
        <section className="bg-white rounded-2xl shadow p-4">
          <div className=\"flex items-center justify-between mb-2\">\n            <h2 className=\"text-xl font-semibold\">Projected Growth</h2>\n            <div className=\"flex items-center gap-3\">\n              <div className=\"text-sm text-neutral-600 hidden md:block\">Start: {fmtUSD(investable)} • Return: {annualReturn}% • Horizon: {years} yrs</div>\n              <Button onClick={exportCSV}>Export CSV</Button>\n            </div>\n          </div>
          </div>
          <div className="h-80 w-full">
            <ResponsiveContainer width="100%" height="100%">
              <LineChart data={data} margin={{ top: 10, right: 20, bottom: 0, left: 0 }}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="year" tickFormatter={(v) => `${v}y`} />
                <YAxis tickFormatter={(v) => `$${(v / 1000).toFixed(0)}k`} />
                <Tooltip
                  formatter={(value: any, name: any) => [fmtUSD(Number(value)), name === "balance" ? "Balance" : "Contributions"]}
                  labelFormatter={(label: any) => `Year ${label}`}
                />
                <Legend />
                <Line type="monotone" dataKey="balance" dot={false} strokeWidth={2} name="Balance" />
                <Line type="monotone" dataKey="contributions" dot={false} strokeDasharray="4 4" name="Contributions" />
              </LineChart>
            </ResponsiveContainer>
          </div>
          <p className="text-xs text-neutral-500 mt-3">Assumes constant return with monthly compounding; illustration only and not investment advice.</p>
        </section>
      </div>
    </div>
  );
}

// === UI helpers ===
function Card({ children }: { children: React.ReactNode }) {
  return <div className="bg-white rounded-2xl shadow p-4 space-y-2">{children}</div>;
}
function Label({ children }: { children: React.ReactNode }) {
  return <label className="text-sm font-medium text-neutral-700">{children}</label>;
}
function SmallLine({ children }: { children: React.ReactNode }) {
  return <p className="text-xs text-neutral-500">{children}</p>;
}
function NumberInput({ value, onChange, step = 1, min = 0, max = 10_000_000, suffix = "" }: {
  value: number;
  onChange: (n: number) => void;
  step?: number;
  min?: number;
  max?: number;
  suffix?: string;
}) {
  return (
    <div className="flex items-center gap-2">
      <input
        type="number"
        className="w-full rounded-lg border border-neutral-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-neutral-400"
        value={Number.isFinite(value) ? value : 0}
        min={min}
        max={max}
        step={step}
        onChange={(e) => onChange(Number(e.target.value))}
      />
      {suffix && <span className="text-sm text-neutral-600 w-8 text-right">{suffix}</span>}
    </div>
  );
}
function Range({ value, onChange, min = 0, max = 100, step = 1 }: {
  value: number;
  onChange: (n: number) => void;
  min?: number;
  max?: number;
  step?: number;
}) {
  return (
    <input
      type="range"
      className="w-full"
      min={min}
      max={max}
      step={step}
      value={Number.isFinite(value) ? value : 0}
      onChange={(e) => onChange(Number(e.target.value))}
    />
  );
}
function StatCard({ title, value, sub }: { title: string; value: string; sub?: string }) {
  return (
    <div className="bg-white rounded-2xl shadow p-4">
      <div className="text-sm text-neutral-600">{title}</div>
      <div className="text-2xl font-semibold">{value}</div>
      {sub && <div className="text-xs text-neutral-500">{sub}</div>}
    </div>
  );
}
# compound-interest-calculator
