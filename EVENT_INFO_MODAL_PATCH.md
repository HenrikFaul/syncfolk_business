import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { X } from 'lucide-react';
import { DatePopoverField } from '@/components/DatePopoverField';

interface BatchVotePanelProps {
  onBatchVote: (startDate: Date, endDate: Date, value: string) => void;
  onClearRangeVotes?: (startDate: Date, endDate: Date) => void;
  onClearAll?: () => void;
  eventStartDate?: string | null;
  eventEndDate?: string | null;
}

export function BatchVotePanel({
  onBatchVote,
  onClearRangeVotes,
  onClearAll,
  eventStartDate,
  eventEndDate,
}: BatchVotePanelProps) {
  const [startDate, setStartDate] = useState<Date>();
  const [endDate, setEndDate] = useState<Date>();
  const [voteValue, setVoteValue] = useState('good');
  const [startCalendarOpen, setStartCalendarOpen] = useState(false);
  const [endCalendarOpen, setEndCalendarOpen] = useState(false);

  const minDate = eventStartDate ? new Date(eventStartDate) : undefined;
  const maxDate = eventEndDate ? new Date(eventEndDate) : undefined;

  const disableBatchDate = (date: Date) => {
    if (minDate && date < minDate) return true;
    if (maxDate && date > maxDate) return true;
    return false;
  };

  return (
    <div className="space-y-4 rounded-2xl border bg-card p-4 shadow-card">
      <h3 className="font-display text-lg font-bold">Batch kitöltés</h3>

      <div className="space-y-2">
        <Label>Szavazat értéke</Label>
        <Select value={voteValue} onValueChange={setVoteValue}>
          <SelectTrigger className="rounded-xl">
            <SelectValue placeholder="Válassz szavazatot..." />
          </SelectTrigger>
          <SelectContent className="rounded-xl">
            <SelectItem value="good">Jó</SelectItem>
            <SelectItem value="maybe">Talán</SelectItem>
            <SelectItem value="bad">Nem jó</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <div className="space-y-2">
        <Label>Kezdő dátum</Label>
        <DatePopoverField
          value={startDate}
          placeholder="Válassz dátumot"
          open={startCalendarOpen}
          onOpenChange={setStartCalendarOpen}
          minDate={minDate}
          onConfirm={(date) => {
            const hadEndDate = !!endDate;
            setStartDate(date);
            if (!endDate || endDate < date) setEndDate(date);
            if (!hadEndDate) setEndCalendarOpen(true);
          }}
        />
      </div>

      <div className="space-y-2">
        <Label>Befejező dátum</Label>
        <DatePopoverField
          value={endDate}
          placeholder="Válassz dátumot"
          open={endCalendarOpen}
          onOpenChange={setEndCalendarOpen}
          minDate={startDate || minDate}
          onConfirm={(date) => setEndDate(date)}
        />
      </div>

      {onClearRangeVotes && (
        <Button
          type="button"
          variant="outline"
          className="w-full rounded-xl"
          disabled={!startDate || !endDate}
          onClick={() => {
            if (!startDate || !endDate) return;
            onClearRangeVotes(startDate, endDate);
          }}
        >
          <X className="mr-2 h-4 w-4" />
          Választások törlése
        </Button>
      )}

      <Button
        type="button"
        className="w-full rounded-xl gradient-primary text-primary-foreground"
        disabled={!startDate || !endDate}
        onClick={() => {
          if (!startDate || !endDate) return;
          onBatchVote(startDate, endDate, voteValue);
        }}
      >
        Alkalmaz
      </Button>

      {onClearAll && (
        <Button type="button" variant="destructive" className="w-full rounded-xl" onClick={onClearAll}>
          Minden szavazatom törlése
        </Button>
      )}
    </div>
  );
}
