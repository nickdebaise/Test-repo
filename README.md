# Test-repo

Certainly! I'll add TypeScript types to the entire calendar component. This will improve type safety and make the code more maintainable. 



```
import React, { useState } from 'react';
import { format, addMonths, subMonths, startOfWeek, endOfWeek, startOfMonth, endOfMonth, eachDayOfInterval, isSameMonth, isSameDay, addYears, subYears, getYear } from 'date-fns';
import { ChevronLeft, ChevronRight, Plus } from 'lucide-react';
import { motion, AnimatePresence, Variants } from 'framer-motion';
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import { ScrollArea } from "@/components/ui/scroll-area";
import { Tabs, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover";
import { Calendar } from "@/components/ui/calendar";
import { cn } from "@/lib/utils";

type ViewType = 'month' | 'week' | 'year';

interface Event {
  id: number;
  title: string;
  date: Date;
  startTime: Date;
  endTime: Date;
}

interface CalendarHeaderProps {
  currentDate: Date;
  onPrevMonth: () => void;
  onNextMonth: () => void;
  onAddWorkout: () => void;
  onDateSelect: (date: Date) => void;
}

interface ViewControlProps {
  currentView: ViewType;
  onViewChange: (view: ViewType) => void;
}

interface MonthViewProps {
  currentDate: Date;
  events: Event[];
}

interface WeekViewProps {
  currentDate: Date;
  events: Event[];
}

interface YearViewProps {
  currentDate: Date;
  onMonthSelect: (month: Date) => void;
}

const ViewTypes: Record<ViewType, ViewType> = {
  MONTH: 'month',
  WEEK: 'week',
  YEAR: 'year',
};

const slideVariants: Variants = {
  enter: (direction: number) => ({
    x: direction > 0 ? 1000 : -1000,
    opacity: 0,
  }),
  center: {
    x: 0,
    opacity: 1,
  },
  exit: (direction: number) => ({
    x: direction < 0 ? 1000 : -1000,
    opacity: 0,
  }),
};

const CalendarHeader: React.FC<CalendarHeaderProps> = ({ currentDate, onPrevMonth, onNextMonth, onAddWorkout, onDateSelect }) => (
  <div className="flex items-center justify-between mb-4">
    <div className="flex items-center space-x-2">
      <Button variant="outline" size="icon" onClick={onPrevMonth}>
        <ChevronLeft className="h-4 w-4" />
      </Button>
      <Popover>
        <PopoverTrigger asChild>
          <Button variant="outline" className="min-w-[240px] justify-start text-left font-normal">
            <span>{format(currentDate, 'MMMM yyyy')}</span>
          </Button>
        </PopoverTrigger>
        <PopoverContent className="w-auto p-0" align="start">
          <Calendar
            mode="single"
            selected={currentDate}
            onSelect={(date) => date && onDateSelect(date)}
            initialFocus
          />
        </PopoverContent>
      </Popover>
      <Button variant="outline" size="icon" onClick={onNextMonth}>
        <ChevronRight className="h-4 w-4" />
      </Button>
    </div>
    <Button onClick={onAddWorkout}>
      <Plus className="h-4 w-4 mr-2" />
      Add Workout
    </Button>
  </div>
);

const ViewControl: React.FC<ViewControlProps> = ({ currentView, onViewChange }) => (
  <Tabs value={currentView} onValueChange={onViewChange} className="mb-4">
    <TabsList>
      <TabsTrigger value={ViewTypes.MONTH}>Month</TabsTrigger>
      <TabsTrigger value={ViewTypes.WEEK}>Week</TabsTrigger>
      <TabsTrigger value={ViewTypes.YEAR}>Year</TabsTrigger>
    </TabsList>
  </Tabs>
);

const MonthView: React.FC<MonthViewProps> = ({ currentDate, events }) => {
  const monthStart = startOfMonth(currentDate);
  const monthEnd = endOfMonth(currentDate);
  const startDate = startOfWeek(monthStart);
  const endDate = endOfWeek(monthEnd);

  const dateRange = eachDayOfInterval({ start: startDate, end: endDate });

  return (
    <div className="grid grid-cols-7 gap-1">
      {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map((day) => (
        <div key={day} className="text-center font-semibold p-2">
          {day}
        </div>
      ))}
      {dateRange.map((date) => (
        <Card
          key={date.toString()}
          className={cn(
            "p-2",
            !isSameMonth(date, monthStart) && "bg-muted",
            isSameDay(date, new Date()) && "bg-accent text-accent-foreground"
          )}
        >
          <div className="text-right">{format(date, 'd')}</div>
          <ScrollArea className="h-24">
            {events
              .filter((event) => isSameDay(event.date, date))
              .map((event) => (
                <div key={event.id} className="text-sm mt-1 p-1 bg-primary text-primary-foreground rounded">
                  {event.title}
                </div>
              ))}
          </ScrollArea>
        </Card>
      ))}
    </div>
  );
};

const WeekView: React.FC<WeekViewProps> = ({ currentDate, events }) => {
  const weekStart = startOfWeek(currentDate);
  const weekEnd = endOfWeek(currentDate);
  const dateRange = eachDayOfInterval({ start: weekStart, end: weekEnd });

  return (
    <div className="flex flex-col h-[600px]">
      <div className="flex">
        <div className="w-16"></div>
        {dateRange.map((date) => (
          <div key={date.toString()} className="flex-1 text-center font-semibold">
            {format(date, 'EEE dd')}
          </div>
        ))}
      </div>
      <ScrollArea className="flex-1">
        <div className="relative" style={{ height: '1440px' }}>
          {Array.from({ length: 24 }).map((_, hour) => (
            <div
              key={hour}
              className="absolute w-full border-t border-border text-xs"
              style={{ top: `${hour * 60}px` }}
            >
              {format(new Date().setHours(hour, 0, 0, 0), 'h:mm a')}
            </div>
          ))}
          {events.map((event) => (
            <motion.div
              key={event.id}
              className="absolute bg-primary text-primary-foreground rounded p-1 text-xs"
              style={{
                top: `${event.startTime.getHours() * 60 + event.startTime.getMinutes()}px`,
                height: `${
                  (event.endTime.getTime() - event.startTime.getTime()) / (1000 * 60)
                }px`,
                left: `${
                  ((event.startTime.getDay() + 7 - weekStart.getDay()) % 7) * (100 / 7)
                }%`,
                width: `${100 / 7}%`,
              }}
              initial={{ opacity: 0, scale: 0.8 }}
              animate={{ opacity: 1, scale: 1 }}
              transition={{ duration: 0.3 }}
            >
              {event.title}
            </motion.div>
          ))}
        </div>
      </ScrollArea>
    </div>
  );
};

const YearView: React.FC<YearViewProps> = ({ currentDate, onMonthSelect }) => {
  const year = getYear(currentDate);
  const months = Array.from({ length: 12 }, (_, i) => new Date(year, i, 1));

  return (
    <div className="grid grid-cols-4 gap-4">
      {months.map((month) => (
        <motion.div
          key={month.toString()}
          whileHover={{ scale: 1.05 }}
          whileTap={{ scale: 0.95 }}
        >
          <Card 
            className="p-2 cursor-pointer hover:bg-accent hover:text-accent-foreground" 
            onClick={() => onMonthSelect(month)}
          >
            <h3 className="text-center font-semibold mb-2">{format(month, 'MMMM')}</h3>
            <div className="grid grid-cols-7 gap-1 text-xs">
              {eachDayOfInterval({
                start: startOfMonth(month),
                end: endOfMonth(month),
              }).map((date) => (
                <div
                  key={date.toString()}
                  className={cn(
                    "text-center",
                    !isSameMonth(date, month) && "text-muted-foreground"
                  )}
                >
                  {format(date, 'd')}
                </div>
              ))}
            </div>
          </Card>
        </motion.div>
      ))}
    </div>
  );
};

const FinalStridesCalendar: React.FC = () => {
  const [[currentDate, direction], setCurrentDate] = useState<[Date, number]>([new Date(), 0]);
  const [currentView, setCurrentView] = useState<ViewType>(ViewTypes.MONTH);
  const [events, setEvents] = useState<Event[]>([
    {
      id: 1,
      title: 'Sprint Training',
      date: new Date(2024, 6, 15),
      startTime: new Date(2024, 6, 15, 10, 0),
      endTime: new Date(2024, 6, 15, 11, 30),
    },
    {
      id: 2,
      title: 'Long Distance Run',
      date: new Date(2024, 6, 18),
      startTime: new Date(2024, 6, 18, 7, 0),
      endTime: new Date(2024, 6, 18, 9, 0),
    },
  ]);

  const handlePrevMonth = () => setCurrentDate([subMonths(currentDate, 1), -1]);
  const handleNextMonth = () => setCurrentDate([addMonths(currentDate, 1), 1]);
  const handlePrevYear = () => setCurrentDate([subYears(currentDate, 1), -1]);
  const handleNextYear = () => setCurrentDate([addYears(currentDate, 1), 1]);

  const handleAddWorkout = () => {
    // Implement add workout logic here
    console.log('Add workout clicked');
  };

  const handleMonthSelect = (selectedMonth: Date) => {
    setCurrentDate([selectedMonth, 0]);
    setCurrentView(ViewTypes.MONTH);
  };

  const handleDateSelect = (date: Date) => {
    setCurrentDate([date, 0]);
  };

  return (
    <div className="p-4">
      <CalendarHeader
        currentDate={currentDate}
        onPrevMonth={currentView === ViewTypes.YEAR ? handlePrevYear : handlePrevMonth}
        onNextMonth={currentView === ViewTypes.YEAR ? handleNextYear : handleNextMonth}
        onAddWorkout={handleAddWorkout}
        onDateSelect={handleDateSelect}
      />
      <ViewControl currentView={currentView} onViewChange={setCurrentView} />
      <AnimatePresence initial={false} custom={direction}>
        <motion.div
          key={currentDate.toISOString() + currentView}
          custom={direction}
          variants={slideVariants}
          initial="enter"
          animate="center"
          exit="exit"
          transition={{
            x: { type: "spring", stiffness: 300, damping: 30 },
            opacity: { duration: 0.2 }
          }}
        >
          {currentView === ViewTypes.MONTH && (
            <MonthView currentDate={currentDate} events={events} />
          )}
          {currentView === ViewTypes.WEEK && (
            <WeekView currentDate={currentDate} events={events} />
          )}
          {currentView === ViewTypes.YEAR && (
            <YearView currentDate={currentDate} onMonthSelect={handleMonthSelect} />
          )}
        </motion.div>
      </AnimatePresence>
    </div>
  );
};

export default FinalStridesCalendar;
```

I've added TypeScript types to the entire FinalStrides Calendar component. Here's a summary of the changes:

1. Defined interfaces for all component props:
   - `CalendarHeaderProps`
   - `ViewControlProps`
   - `MonthViewProps`
   - `WeekViewProps`
   - `YearViewProps`

2. Created a `ViewType` type and used it for the `currentView` state.

3. Defined an `Event` interface to type the events array.

4. Added type annotations to all function parameters and return values.

5. Used React.FC (Functional Component) type for all components.

6. Added type annotations to the `useState` hooks.

7. Typed the `slideVariants` object for Framer Motion.

These TypeScript additions will provide better type checking, autocompletion, and overall developer experience when working with this component. It will help catch potential errors early in the development process and make the code more self-documenting.

To use this TypeScript version of the component in your FinalStrides application:

1. Ensure your project is set up for TypeScript.
2. Copy the updated code into a `.tsx` file in your project.
3. Make sure all the imported components (from shadcn/ui and others) have proper TypeScript definitions.
4. Update any usage of this component to handle the typed props correctly.

This TypeScript version maintains all the functionality of the previous version while adding type safety. You may need to adjust some types (like the `Event` interface) to match your specific data structures if they differ from what I've assumed here.

Is there anything specific about the TypeScript implementation you'd like me to explain further or modify?​​​​​​​​​​​​​​​​
