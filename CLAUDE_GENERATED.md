# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Sun Jul 27 15:42:28 UTC 2025

## Generated Content

```
I'll help you implement the requested features for your Ionic React TypeScript government billing/invoicing application. Based on the codebase analysis, I can see this is a spreadsheet-based invoicing system using SocialCalc. Let me implement the key features you mentioned.

## FILES_TO_CREATE

1. `src/components/RoleGuard/RoleGuard.tsx`
2. `src/hooks/useAuth.ts`
3. `src/types/auth.ts`
4. `src/utils/pdfExport.ts`
5. `src/hooks/useI18n.ts`
6. `src/translations/index.ts`
7. `src/translations/en.ts`
8. `src/translations/es.ts`
9. `src/hooks/useTheme.ts`
10. `src/components/ExportButton/ExportButton.tsx`
11. `src/components/ThemeToggle/ThemeToggle.tsx`
12. `src/components/LanguageSelector/LanguageSelector.tsx`
13. `src/components/RoleGuard/RoleGuard.test.tsx`
14. `src/utils/__tests__/pdfExport.test.ts`

## FILES_TO_MODIFY

1. `src/App/App.js` → Convert to TypeScript and add new features
2. `src/index.js` → Add theme and auth providers
3. `src/Menu/Menu.js` → Add role-based menu items
4. `package.json` → Add new dependencies

## CODE_CHANGES

### 1. Update package.json
```json
{
  "name": "react-webapp",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@tanstack/react-query": "^5.49.2",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "@web3-storage/w3up-client": "^17.1.2",
    "@ionic/react": "^7.0.0",
    "@ionic/react-router": "^7.0.0",
    "connectkit": "^1.8.2",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router": "^6.0.0",
    "react-router-dom": "^6.0.0",
    "react-scripts": "5.0.1",
    "viem": "2.x",
    "wagmi": "^2.10.9",
    "web-vitals": "^2.1.4",
    "whatwg-fetch": "^2.0.3",
    "jspdf": "^2.5.1",
    "html2canvas": "^1.4.1",
    "i18next": "^23.0.0",
    "react-i18next": "^13.0.0"
  },
  "devDependencies": {
    "@babel/plugin-proposal-private-property-in-object": "^7.21.11",
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@types/node": "^18.0.0",
    "typescript": "^5.0.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

### 2. Create src/types/auth.ts
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  permissions: Permission[];
  createdAt: string;
  lastLogin?: string;
}

export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  ACCOUNTANT = 'accountant',
  VIEWER = 'viewer'
}

export enum Permission {
  CREATE_INVOICE = 'create_invoice',
  EDIT_INVOICE = 'edit_invoice',
  DELETE_INVOICE = 'delete_invoice',
  VIEW_INVOICE = 'view_invoice',
  EXPORT_INVOICE = 'export_invoice',
  MANAGE_USERS = 'manage_users',
  VIEW_REPORTS = 'view_reports',
  MANAGE_SETTINGS = 'manage_settings'
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthContextType extends AuthState {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  hasPermission: (permission: Permission) => boolean;
  hasRole: (role: UserRole) => boolean;
}
```

### 3. Create src/hooks/useAuth.ts
```typescript
import { createContext, useContext, useReducer, useCallback, ReactNode } from 'react';
import { AuthState, AuthContextType, LoginCredentials, User, UserRole, Permission } from '../types/auth';

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
};

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'CLEAR_ERROR' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: true,
        user: action.payload,
        error: null,
      };
    case 'LOGIN_FAILURE':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: false,
        user: null,
        error: action.payload,
      };
    case 'LOGOUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null,
        error: null,
      };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
    default:
      return state;
  }
};

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  const login = useCallback(async (credentials: LoginCredentials) => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      // Mock authentication - replace with actual API call
      const mockUser: User = {
        id: '1',
        email: credentials.email,
        name: 'John Doe',
        role: UserRole.ADMIN,
        permissions: [
          Permission.CREATE_INVOICE,
          Permission.EDIT_INVOICE,
          Permission.DELETE_INVOICE,
          Permission.VIEW_INVOICE,
          Permission.EXPORT_INVOICE,
          Permission.MANAGE_USERS,
          Permission.VIEW_REPORTS,
          Permission.MANAGE_SETTINGS,
        ],
        createdAt: new Date().toISOString(),
        lastLogin: new Date().toISOString(),
      };

      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      localStorage.setItem('authToken', 'mock-token');
      localStorage.setItem('user', JSON.stringify(mockUser));
      
      dispatch({ type: 'LOGIN_SUCCESS', payload: mockUser });
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: 'Authentication failed' });
    }
  }, []);

  const logout = useCallback(() => {
    localStorage.removeItem('authToken');
    localStorage.removeItem('user');
    dispatch({ type: 'LOGOUT' });
  }, []);

  const hasPermission = useCallback((permission: Permission): boolean => {
    return state.user?.permissions.includes(permission) || false;
  }, [state.user]);

  const hasRole = useCallback((role: UserRole): boolean => {
    return state.user?.role === role || false;
  }, [state.user]);

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    hasPermission,
    hasRole,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### 4. Create src/components/RoleGuard/RoleGuard.tsx
```typescript
import React from 'react';
import { IonAlert } from '@ionic/react';
import { useAuth } from '../../hooks/useAuth';
import { UserRole, Permission } from '../../types/auth';

interface RoleGuardProps {
  children: React.ReactNode;
  requiredRole?: UserRole;
  requiredPermission?: Permission;
  fallback?: React.ReactNode;
  showAlert?: boolean;
}

export const RoleGuard: React.FC<RoleGuardProps> = ({
  children,
  requiredRole,
  requiredPermission,
  fallback,
  showAlert = false,
}) => {
  const { user, hasRole, hasPermission } = useAuth();
  const [showAccessDenied, setShowAccessDenied] = React.useState(false);

  if (!user) {
    return fallback ? <>{fallback}</> : null;
  }

  const hasRequiredRole = requiredRole ? hasRole(requiredRole) : true;
  const hasRequiredPermission = requiredPermission ? hasPermission(requiredPermission) : true;

  if (!hasRequiredRole || !hasRequiredPermission) {
    if (showAlert) {
      setShowAccessDenied(true);
    }
    return (
      <>
        {fallback ? <>{fallback}</> : null}
        <IonAlert
          isOpen={showAccessDenied}
          onDidDismiss={() => setShowAccessDenied(false)}
          header="Access Denied"
          message="You don't have permission to access this feature."
          buttons={['OK']}
        />
      </>
    );
  }

  return <>{children}</>;
};
```

### 5. Create src/utils/pdfExport.ts
```typescript
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

export interface ExportOptions {
  filename?: string;
  format?: 'a4' | 'letter';
  orientation?: 'portrait' | 'landscape';
  quality?: number;
  margin?: number;
}

export const exportToPDF = async (
  elementId: string,
  options: ExportOptions = {}
): Promise<void> => {
  const {
    filename = 'invoice.pdf',
    format = 'a4',
    orientation = 'portrait',
    quality = 1,
    margin = 10,
  } = options;

  try {
    const element = document.getElementById(elementId);
    if (!element) {
      throw new Error(`Element with id "${elementId}" not found`);
    }

    // Create canvas from the element
    const canvas = await html2canvas(element, {
      scale: quality,
      useCORS: true,
      logging: false,
      backgroundColor: '#ffffff',
    });

    // Calculate dimensions
    const imgWidth = format === 'a4' ? 210 : 216; // A4 or Letter width in mm
    const imgHeight = format === 'a4' ? 297 : 279; // A4 or Letter height in mm
    const canvasImgWidth = imgWidth - (margin * 2);
    const canvasImgHeight = (canvas.height * canvasImgWidth) / canvas.width;

    // Create PDF
    const pdf = new jsPDF({
      orientation,
      unit: 'mm',
      format: format === 'a4' ? 'a4' : 'letter',
    });

    // Add image to PDF
    const imgData = canvas.toDataURL('image/png');
    
    let heightLeft = canvasImgHeight;
    let position = margin;

    // Add first page
    pdf.addImage(imgData, 'PNG', margin, position, canvasImgWidth, canvasImgHeight);
    heightLeft -= (imgHeight - (margin * 2));

    // Add additional pages if needed
    while (heightLeft >= 0) {
      position = heightLeft - canvasImgHeight + margin;
      pdf.addPage();
      pdf.addImage(imgData, 'PNG', margin, position, canvasImgWidth, canvasImgHeight);
      heightLeft -= (imgHeight - (margin * 2));
    }

    // Save the PDF
    pdf.save(filename);
  } catch (error) {
    console.error('Error exporting to PDF:', error);
    throw new Error('Failed to export PDF');
  }
};

export const exportInvoiceToPDF = async (invoiceData: any): Promise<void> => {
  const filename = `invoice_${invoiceData.invoiceNumber || 'draft'}_${new Date().toISOString().split('T')[0]}.pdf`;
  
  await exportToPDF('tableeditor', {
    filename,
    format: 'a4',
    orientation: 'portrait',
    quality: 2,
    margin: 15,
  });
};
```

### 6. Create src/translations/index.ts
```typescript
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import en from './en';
import es from './es';

const resources = {
  en: { translation: en },
  es: { translation: es },
};

i18n
  .use(initReactI18next)
  .init({
    resources,
    lng: 'en',
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false,
    },
  });

export default i18n;
```

### 7. Create src/translations/en.ts
```typescript
export default {
  common: {
    save: 'Save',
    cancel: 'Cancel',
    delete: 'Delete',
    edit: 'Edit',
    create: 'Create',
    export: 'Export',
    loading: 'Loading...',
    error: 'Error',
    success: 'Success',
    confirm: 'Confirm',
    yes: 'Yes',
    no: 'No',
  },
  app: {
    title: 'Invoice Suite',
    darkMode: 'Dark Mode',
    lightMode: 'Light Mode',
    language: 'Language',
  },
  auth: {
    login: 'Login',
    logout: 'Logout',
    email: 'Email',
    password: 'Password',
    loginError: 'Login failed. Please check your credentials.',
    accessDenied: 'Access Denied',
    noPermission: "You don't have permission to access this feature.",
  },
  invoice: {
    title: 'Invoice',
    number: 'Invoice Number',
    date: 'Date',
    billTo: 'Bill To',
    from: 'From',
    description: 'Description',
    amount: 'Amount',
    total: 'Total',
    subtotal: 'Subtotal',
    tax: 'Tax',
    taxRate: 'Tax Rate',
    create: 'Create Invoice',
    edit: 'Edit Invoice',
    delete: 'Delete Invoice',
    export: 'Export Invoice',
    exportToPDF: 'Export to PDF',
    exportSuccess: 'Invoice exported successfully',
    exportError: 'Failed to export invoice',
  },
  menu: {
    home: 'Home',
    invoices: 'Invoices',
    reports: 'Reports',
    settings: 'Settings',
    users: 'Users',
    profile: 'Profile',
  },
  roles: {
    admin: 'Administrator',
    manager: 'Manager',
    accountant: 'Accountant',
    viewer: 'Viewer',
  },
};
```

### 8. Create src/translations/es.ts
```typescript
export default {
  common: {
    save: 'Guardar',
    cancel: 'Cancelar',
    delete: 'Eliminar',
    edit: 'Editar',
    create: 'Crear',
    export: 'Exportar',
    loading: 'Cargando...',
    error: 'Error',
    success: 'Éxito',
    confirm: 'Confirmar',
    yes: 'Sí',
    no: 'No',
  },
  app: {
    title: 'Suite de Facturación',
    darkMode: 'Modo Oscuro',
    lightMode: 'Modo Claro',
    language: 'Idioma',
  },
  auth: {
    login: 'Iniciar Sesión',
    logout: 'Cerrar Sesión',
    email: 'Correo Electrónico',
    password: 'Contraseña',
    loginError: 'Error de inicio de sesión. Verifique sus credenciales.',
    accessDenied: 'Acceso Denegado',
    noPermission: 'No tiene permisos para acceder a esta función.',
  },
  invoice: {
    title: 'Factura',
    number: 'Número de Factura',
    date: 'Fecha',
    billTo: 'Facturar A',
    from: 'De',
    description: 'Descripción',
    amount: 'Cantidad',
    total: 'Total',
    subtotal: 'Subtotal',
    tax: 'Impuesto',
    taxRate: 'Tasa de Impuesto',
    create: 'Crear Factura',
    edit: 'Editar Factura',
    delete: 'Eliminar Factura',
    export: 'Exportar Factura',
    exportToPDF: 'Exportar a PDF',
    exportSuccess: 'Factura exportada exitosamente',
    exportError: 'Error al exportar la factura',
  },
  menu: {
    home: 'Inicio',
    invoices: 'Facturas',
    reports: 'Informes',
    settings: 'Configuración',
    users: 'Usuarios',
    profile: 'Perfil',
  },
  roles: {
    admin: 'Administrador',
    manager: 'Gerente',
    accountant: 'Contador',
    viewer: 'Observador',
  },
};
```

### 9. Create src/hooks/useI18n.ts
```typescript
import { useTranslation } from 'react-i18next';

export const useI18n = () => {
  const { t, i18n } = useTranslation();

  const changeLanguage = (lang: string) => {
    i18n.changeLanguage(lang);
    localStorage.setItem('preferredLanguage', lang);
  };

  const getCurrentLanguage = () => i18n.language;

  const getAvailableLanguages = () => [
    { code: 'en', name: 'English' },
    { code: 'es', name: 'Español' },
  ];

  return {
    t,
    changeLanguage,
    getCurrentLanguage,
    getAvailableLanguages,
  };
};
```

### 10. Create src/hooks/useTheme.ts
```typescript
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>(() => {
    const savedTheme = localStorage.getItem('theme') as Theme;
    return savedTheme || 'light';
  });

  useEffect(() => {
    document.body.classList.remove('dark', 'light');
    document.body.classList.add(theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setThemeState(prev => prev === 'light' ? 'dark' : 'light');
  };

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 11. Create src/components/ExportButton/ExportButton.tsx
```typescript
import React, { useState } from 'react';
import {
  IonButton,
  IonIcon,
  IonActionSheet,
  IonToast,
  IonLoading,
} from '@ionic/react';
import { downloadOutline, documentOutline } from 'ionicons/icons';
import { useI18n } from '../../hooks/useI18n';
import { exportInvoiceToPDF } from '../../utils/pdfExport';

interface ExportButtonProps {
  invoiceData?: any;
  disabled?: boolean;
}

export const ExportButton: React.FC<ExportButtonProps> = ({
  invoiceData,
  disabled = false,
}) => {
  const { t } = useI18n();
  const [showActionSheet, setShowActionSheet] = useState(false);
  const [showToast, setShowToast] = useState(false);
  const [toastMessage, setToastMessage] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleExportToPDF = async () => {
    setIsLoading(true);
    try {
      await exportInvoiceToPDF(invoiceData || {});
      setToastMessage(t('invoice.exportSuccess'));
      setShowToast(true);
    } catch (error) {
      console.error('Export error:', error);
      setToastMessage(t('invoice.exportError'));
      setShowToast(true);
    } finally {
      setIsLoading(false);
    }
  };

  const exportOptions = [
    {
      text: t('invoice.exportToPDF'),
      icon: documentOutline,
      handler: handleExportToPDF,
    },
  ];

  return (
    <>
      <IonButton
        fill="outline"
        onClick={() => setShowActionSheet(true)}
        disabled={disabled}
      >
        <IonIcon icon={downloadOutline} slot="start" />
        {t('common.export')}
      </IonButton>

      <IonActionSheet
        isOpen={showActionSheet}
        onDidDismiss={() => setShowActionSheet(false)}
        buttons={[
          ...exportOptions,
          {
            text: t('common.cancel'),
            role: 'cancel',
          },
        ]}
      />

      <IonLoading isOpen={isLoading} message={t('common.loading')} />

      <IonToast
        isOpen={showToast}
        onDidDismiss={() => setShowToast(false)}
        message={toastMessage}
        duration={3000}
        position="top"
      />
    </>
  );
};
```

### 12. Create src/components/ThemeToggle/ThemeToggle.tsx
```typescript
import React from 'react';
import { IonButton, IonIcon } from '@ionic/react';
import { moonOutline, sunnyOutline } from 'ionicons/icons';
import { useTheme } from '../../hooks/useTheme';
import { useI18n } from '../../hooks/useI18n';

export const ThemeToggle: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  const { t } = useI18n();

  return (
    <IonButton
      fill="clear"
      onClick={toggleTheme}
      aria-label={theme === 'dark' ? t('app.lightMode') : t('app.darkMode')}
    >
      <IonIcon
        icon={theme === 'dark' ? sunnyOutline : moonOutline}
        slot="icon-only"
      />
    </IonButton>
  );
};
```

### 13. Create src/components/LanguageSelector/LanguageSelector.tsx
```typescript
import React, { useState } from 'react';
import {
  IonButton,
  IonIcon,
  IonActionSheet,
  IonItem,
  IonLabel,
} from '@ionic/react';
import { languageOutline } from 'ionicons/icons';
import { useI18n } from '../../hooks/useI18n';

export const LanguageSelector: React.FC = () => {
  const { t, changeLanguage, getCurrentLanguage, getAvailableLanguages } = useI18n();
  const [showActionSheet, setShowActionSheet] = useState(false);

  const languages = getAvailableLanguages();
  const currentLang = getCurrentLanguage();

  const languageButtons = languages.map(lang => ({
    text: lang.name,
    handler: () => changeLanguage(lang.code),
    cssClass: currentLang === lang.code ? 'selected' : '',
  }));

  return (
    <>
      <IonButton
        fill="clear"
        onClick={() => setShowActionSheet(true)}
        aria-label={t('app.language')}
      >
        <IonIcon icon={languageOutline} slot="start" />
        {languages.find(lang => lang.code === currentLang)?.name}
      </IonButton>

      <IonActionSheet
        isOpen={showActionSheet}
        onDidDismiss={() => setShowActionSheet(false)}
        header={t('app.language')}
        buttons={[
          ...languageButtons,
          {
            text: t('common.cancel'),
            role: 'cancel',
          },
        ]}
      />
    </>
  );
};
```

### 14. Update src/index.js → src/index.tsx
```typescript
import React from "react";
import App from "./App/App";
import "./index.css";
import { createRoot } from "react-dom/client";
import { Web3Provider } from "./utils/Web3Provider";
import { AuthProvider } from "./hooks/useAuth";
import { ThemeProvider } from "./hooks/useTheme";
import "./translations";

const domNode = document.getElementById("root");
const root = createRoot(domNode);
root.render(
  <ThemeProvider>
    <AuthProvider>
      <Web3Provider>
        <App />
      </Web3Provider>
    </AuthProvider>
  </ThemeProvider>
);
```

### 15. Update src/App/App.js → src/App/App.tsx
```typescript
import React, { Component } from "react";
import "./App.css";
import * as AppGeneral from "../socialcalc/AppGeneral";
import { DATA } from "../app-data.js";
import { IonApp, IonContent, IonHeader, IonToolbar, IonTitle, IonButtons } from '@ionic/react';
import { setupIonicReact } from '@ionic/react';

import Menu from "../Menu/Menu";
import Files from "../Files/Files";
import { ConnectKitButton } from "connectkit";
import { RoleGuard } from "../components/RoleGuard/RoleGuard";
import { ExportButton } from "../components/ExportButton/ExportButton";
import { ThemeToggle } from "../components/ThemeToggle/ThemeToggle";
import { LanguageSelector } from "../components/LanguageSelector/LanguageSelector";
import { useAuth } from "../hooks/useAuth";
import { useI18n } from "../hooks/useI18n";
import { Permission } from "../types/auth";

setupIonicReact();

interface AppState {
  selectedFile: string;
  device: string;
  listFiles: boolean;
}

class App extends Component<{}, AppState> {
  constructor(props: {}) {
    super(props);
    this.state = {
      selectedFile: "default",
      device: AppGeneral.getDeviceType(),
      listFiles: false,
    };
    this.updateSelectedFile = this.updateSelectedFile.bind(this);
    this.toggleListFiles = this.toggleListFiles.bind(this);
  }

  updateSelectedFile(selectedFile: string) {
    this.setState({
      selectedFile: selectedFile,
    });
  }

  toggleListFiles() {
    this.setState((prevState) => ({
      listFiles: !prevState.listFiles,
    }));
  }

  componentDidMount() {
    const { device, selectedFile } = this.state;
    const data = DATA.ledger[device] || DATA.ledger.default;
    AppGeneral.initializeApp(data);
  }

  componentDidUpdate(prevProps: {}, prevState: AppState) {
    if (prevState.selectedFile !== this.state.selectedFile) {
      const { device, selectedFile } = this.state;
      const data = DATA.ledger[device] || DATA.ledger.default;
      AppGeneral.initializeApp(data);
    }
  }

  render() {
    const { selectedFile, device, listFiles } = this.state;
    const currentData = DATA.ledger[device] || DATA.ledger.default;

    return (
      <IonApp>
        <IonHeader>
          <IonToolbar>
            <IonTitle>Invoice Suite</IonTitle>
            <IonButtons slot="end">
              <RoleGuard requiredPermission={Permission.EXPORT_INVOICE}>
                <ExportButton />
              </RoleGuard>
              <LanguageSelector />
              <ThemeToggle />
              <ConnectKitButton />
            </IonButtons>
          </Ion
```
